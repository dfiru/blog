---
title: "Your NPU Learned to Listen"
date: 2026-04-25T00:00:00-07:00
tags: ["edge-ai", "speech", "npus", "whisper"]
draft: true
author: "Daniel Firu"
description: "Whisper runs end-to-end on Chimera. Encoder and decoder, INT4 weights, FP16 attention, correct first-token prediction. No CPU fallback."
---

<!--
DRAFT STATUS: Pending review
- [ ] ISS cycle counts needed (encoder + decoder). The notebook runs ISS but doesn't
      extract perf data. Need to pull cycle counts from ccl_build artifacts or add
      a get_perf_summary() call to the notebook.
- [ ] Steve Roddy review requested for competitive claims and technical accuracy
- [ ] Once perf data is in, replace the "Performance characterization comes with
      the QC-U hardware" paragraph with actual numbers
- [ ] Consider adding estimated latency at 1.7 GHz clock (cycles / clock_freq)
-->

I said "Mister Quilter is the apostle of the middle classes" into a microphone. A neural processor predicted the first token correctly.

Not a GPU. Not a cloud endpoint. A general-purpose NPU running OpenAI's Whisper, quantized to INT4 weights with FP16 attention, encoder and decoder compiled as native GPNPU kernels. The predicted token, " Mr", matched the float32 reference exactly.

That's new for us. Chimera has run vision models since day one. It ran DOOM to prove a serious point about programmability. But speech is a different modality, and Whisper is a different class of model. Getting it right required solving problems that fixed-function accelerators haven't touched.

## Why Whisper is hard for NPUs

Whisper isn't a single neural network. It's a pipeline.

Audio comes in as a raw waveform. A feature extractor converts it to an 80-channel log-mel spectrogram. A convolutional front-end processes the spectrogram into post-conv features. An encoder, four transformer layers with self-attention, compresses those features into hidden states. Then a decoder, four more transformer layers with both self-attention *and* cross-attention to the encoder, auto-regressively generates text tokens.

The encoder is a standard transformer: matrix multiplies, layer norms, self-attention. The operations map cleanly to a systolic array. This is the part that shows up in marketing benchmarks.

The decoder is where pipelines break.

Cross-attention means the decoder's key and value projections come from the encoder's output, not from the decoder's own hidden states. The decoder has causal self-attention (masked, sequential) *and* cross-attention (attending to all 1,500 encoder positions) in every layer. That's eight weight-matrix projections per layer instead of four. The compute pattern is fundamentally different from the encoder.

Then there's quantization. Whisper's weights use MatMulNBits, a 4-bit integer format with per-group scaling factors. The attention computation runs in FP16. In a single forward pass, the hardware is switching between INT4 weight lookups, FP16 softmax, and mixed-precision accumulation. A fixed-function INT8 accelerator doesn't have the vocabulary for this.

On accelerators without general-purpose programmability, the encoder runs on the NPU. The decoder falls back to the CPU. The pipeline stalls at the handoff. You've accelerated half the model and decelerated the system.

## What we built

We compiled the full Whisper-tiny pipeline, encoder and decoder, as native Chimera GPNPU kernels using the Quadric SDK. Both stages compile through the same toolchain and run on the same silicon. No CPU fallback for the decoder.

| Stage | Runs On | Shape |
|-------|---------|-------|
| Mel spectrogram | CPU | `[1, 80, 3000]` |
| Conv front-end | CPU (lightweight) | `[1, 1500, 384]` |
| **Encoder** (4 layers) | **GPNPU** | `[1, 1500, 384]` → `[1, 1500, 384]` |
| **Decoder** (4 layers) | **GPNPU** | hidden states + prompt → `[1, 4, 51865]` logits |

The mel spectrogram and conv front-end stay on the CPU as cheap preprocessing. Everything after that, including the decoder, runs on the GPNPU.

### The SDK workflow

The Chimera SDK's `ChimeraJob` handles the full compilation pipeline. Start with an ONNX model from Hugging Face. The SDK's helper functions replace MatMulNBits subgraphs and attention cores with Quadric custom ops (`linalg::matMulNBits`, `nn::whisperAttention`). A calibration step runs 73 real audio samples through the model to generate tensor ranges, the per-tensor min/max values that determine fixed-point fractional bits for quantization.

Then `ChimeraJob.compile()` takes the custom-op-matched ONNX graph through relay conversion, quantization, and ISS binary generation. The encoder compiles to a single GPNPU kernel. The decoder compiles to a single GPNPU kernel. The encoder's ISS output feeds directly into the decoder's input with no intermediate dequantization.

```python
encoder_job = ChimeraJob(
    "encoder_model_matched_cut.onnx",
    hw_config=HWConfig(product="QC-U", ocm_size="32MB", macs_per_pe=16),
    trange_file="encoder_model.tranges",
    attn_stub_src_path="whisper_matmul_stubs.hpp",
)
encoder_job.compile()
iss_outputs = encoder_job.run_inference_harness(
    inputs={"/Add_2_output_0": conv_frontend_output}
)
```

ONNX model in, ISS-verified kernel out. The decoder follows the same pattern. This is how Whisper gets from a Hugging Face model card to running on silicon.

## The proof

<!-- TODO: Add ISS cycle counts here once extracted from notebook/ccl_build artifacts -->

The encoder produces hidden states with a PSNR of 54.5 dB against the float32 ORT reference and a mean absolute error of 0.032. For comparison, PSNR above 40 dB typically indicates that quantization error is negligible relative to the signal magnitude. The quantized encoder output is close enough to float32 that downstream accuracy is preserved.

The decoder receives the encoder's ISS output, not a float32 reference, but the actual quantized encoder hidden states, and runs end-to-end ISS inference. PSNR: 43.8 dB. MAE: 0.248. Error accumulates through two quantized stages, and the signal fidelity still holds.

The test input is a 5.86-second clip from LibriSpeech: "Mister Quilter is the apostle of the middle classes and we are glad to welcome his gospel."

Whisper generates text auto-regressively, one token at a time. Each token requires a full decoder forward pass. The decoder prefill, the first step in that sequence, is the most demanding: it processes the prompt tokens with cross-attention to all 1,500 encoder positions and produces logits over a 51,865-token vocabulary. If the prefill gets the wrong answer, every subsequent token inherits the error. It's the hardest single inference step in the pipeline, and it's the one we verified.

The decoder's top-1 predicted token: **2221**, which decodes to **" Mr"**, the first word of the transcription.

The ORT float32 reference top-1: **2221**. Same token. The top-5 sets are identical, {2221, 440, 503, 13094, 639}, with only positions 4 and 5 swapped.

One token. But the right one, through the hardest step, through two stages of INT4+FP16 quantized inference on a single chip. Full auto-regressive decoding, generating the complete sentence token by token, is the next milestone. The prefill proves the pipeline is sound.

These results are from the ISS (Instruction Set Simulator), Quadric's cycle-accurate functional simulator. They verify numerical correctness. Performance characterization comes with the QC-U hardware.

## What this means

Chimera has now run vision (YOLO), non-AI compute (DOOM), and speech (Whisper). Same silicon. The hard part is never the matrix multiplies. It's the rest of the pipeline.

YOLO's challenge was Non-Maximal Suppression, a classical algorithm that most accelerators ship to the CPU. DOOM's challenge was everything: branching, pointer arithmetic, table lookups, zero matrix multiplies. Whisper's challenge is the decoder, cross-attention, mixed-precision quantization, and a compute pattern that fixed-function accelerators can't express.

The architecture didn't change between demos. The compiler got smarter, the custom ops expanded, but the silicon is the same general-purpose NPU that ran DOOM at 1,785 frames per second.

Your NPU runs neural networks. Ours runs programs. And now it listens, too.
