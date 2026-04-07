---
title: "The Next Application Processor Will Be Mostly NPU"
date: 2026-04-07T17:00:00-07:00
tags: ["edge-ai", "semiconductors", "npus"]
draft: false
author: "Daniel Firu"
description: "The die area ratio is flipping. CPUs are shrinking, NPUs are eating the silicon budget, and the application processor you use in 3 years will look nothing like the one you use today."
---

I got rate-limited by my AI provider three times this week. Each time, I stopped what I was doing, waited, and came back when the meter reset. I pay for a premium tier. It didn't matter.

That experience is the entire argument for where silicon is going.

## The demand for intelligence is infinite

The phrase I keep coming back to is "more with more." Not "do more with less" — that's the old efficiency story. The new story is that once you give people access to intelligence, they want unfettered access to it. They want it local. They want it always available. They want it on the laptop, the workstation, the phone. Not behind an API that throttles you during peak hours.

I build neural processor IP for a living. Since 2017, my team at [Quadric](https://quadric.ai) has been designing general-purpose NPUs for edge inference. For most of that time, the pitch was about efficiency and power budgets. Important, but abstract. Now the pitch is personal: I want my own compute because I'm tired of being cut off from it.

## Where we are today

Look at any modern mobile SoC. The NPU is there, but it's a supporting actor. On a typical application processor today, the NPU might occupy 10-15% of the die area. The CPU complex and GPU still dominate the silicon budget. The NPU handles specific tasks — wake word detection, camera ISP enhancement, maybe some on-device transcription — but it's not the main character.

That's about to change.

## The ratio flips

Within 3-5 years, the NPU will consume over 80% of the die area on a mainstream application processor. The CPU shrinks to a coordination role. The GPU consolidates or gets absorbed. The NPU becomes the processor, and everything else becomes overhead.

The forcing function isn't one thing. It's the convergence of several:

**Models are getting smaller and more capable.** A 1-bit quantized 8B parameter model now runs at 40 tokens per second on an iPhone. Two years ago that was a cloud-only workload. The models aren't just shrinking, they're getting good enough to be useful at sizes that fit on a phone.

**The workloads demand it.** Every app is becoming an AI app. Not in the marketing sense, in the architectural sense. The camera pipeline, the keyboard, the search bar, the notification system, the accessibility stack — all of these are becoming inference workloads. When everything is inference, the processor better be built for inference.

**People want local.** Privacy. Control. Auditability. And fidelity — cloud providers quietly run degraded model versions during peak hours. You're paying for GPT-5 and getting GPT-5-mini when the datacenter is busy. Local means you get what you get, every time, no throttling, no quality degradation.

## The cloud counterargument

Someone will point out that cloud inference keeps getting cheaper. This is true and irrelevant. Cloud inference also keeps getting rate-limited, degraded, surveilled, and interrupted. The cost curve is not the only curve that matters.

I pay hundreds of dollars a month for AI tooling. This week I hit my usage cap and had to stop working. My enterprise account has no token usage analytics in the backend. The percentage meter is, as far as I can tell, made up. The renewal will certainly cost more.

The real cost of cloud inference isn't the price per token. It's the cost of not having it when you need it.

## What this means for silicon

If you're designing an application processor today, you have a choice. You can keep the NPU as a 15% sidecar and ship a chip that's already obsolete by the time it hits production. Or you can bet on the ratio flip and design for a world where the NPU is the processor.

This means rethinking the memory architecture. It means rethinking the interconnect. It means rethinking what the CPU even does when 80% of the compute is happening on the NPU. Most importantly, it means the NPU can't be a fixed-function accelerator that only runs a handful of blessed operators. It has to be general-purpose. It has to run the full network, pre-processing and post-processing included, without falling back to the CPU for the hard parts.

CPU fallback is the original sin of NPU design. When the NPU can't handle an operator, it ships it to the CPU, which runs it 100x slower, and the whole pipeline stalls. At 15% die area, you can tolerate some fallback. At 80% die area, fallback means you built the wrong chip.

## The sign is clear

I posted this on X last week and it's the most engagement I've gotten on a substantive take: "The signs for the future are clear. Your main application processor will be mostly NPU and very little CPU."

The responses confirmed what I already knew. People feel this. Not because they read a whitepaper, but because they got rate-limited, or their on-device transcription finally got good enough, or they ran a 7B model on their laptop and realized they don't need the cloud for most of what they do.

The silicon will follow the demand. It always does. The question is whether your NPU is ready for 80% of the die.
