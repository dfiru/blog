+++
title = "Introducing Simulchip: Streamlining Netrunner Proxy Management"
date = "2025-07-14T00:00:00-00:00"
author = "Daniel Firu"
authorTwitter = "danfiru"
cover = ""
tags = ["technology", "gaming", "python", "open-source"]
keywords = ["netrunner", "card-game", "proxy", "python", "cli"]
description = "The story behind Simulchip - a Python tool I built to solve the practical challenge of managing card collections and generating proxy cards for the Netrunner TCG."
showFullContent = false
readingTime = false
hideComments = false
color = ""
+++

# The Problem: Building a Netrunner Collection from Scratch in 2025

When I recently dove into Netrunner after purchasing System Gateway and Elevation around the Elevation release date, I quickly discovered what every new player learns: you need a lot of cards to play Standard at a local. Playing on jinteki.net is absolutely fine, but as my interest continued, I wondered if there were any local players in my area. And sure enough, there were. But where was I going to get 6 packs worth of cards from? What if I end up not liking it that much? Proxying the cards in the meantime seemed like a good idea.

The traditional approach involves manually tracking your collection and copy-pasting missing cards on proxy generating sites. Or finding the right pages you need from pack PDFs released by Null Signal Games—printing out only the specific pages you need. Being an engineer, I can't let any such process not be procedurally automated.

Enter Simulchip.

## The Solution: Automated Collection Management

Simulchip is a Python library and CLI tool that integrates directly with NetrunnerDB to solve the proxy management problem elegantly. The core insight was simple: if the data exists in a structured format (which it does, thanks to NetrunnerDB's excellent API), then collection management and proxy generation should be automated, not manual. We tell Simulchip which cards we have, we point it to a net deck on NetrunnerDB, and out comes a PDF.

### Key Features

The tool provides several core capabilities:

- **Collection Management**: Interactive tracking of owned cards with filtering and search functionality
- **Proxy Generation**: Automated creation of print-ready PDF sheets with precise card sizing (63mm x 88mm)
- **Convenient Output**: Generated PDFs include cut guidelines for clean, tournament-legal proxies

## Technical Approach

Simulchip is built on Python 3.10+. I figure the intersection of people who play Netrunner and can install a Python package is pretty decent.

- `requests` for NetrunnerDB API integration
- `reportlab` for PDF generation with precise measurements
- `Pillow` for image processing and layout
- `typer` for the command line interface

The tool supports both programmatic usage as a library and direct CLI interaction.

### Example Usage

Here's a simple example:

```bash
# Generate proxies for missing cards in a deck
simulchip proxy "https://netrunnerdb.com/en/decklist/8b0bb2b5-840a-423b-af55-bfe73e0e012d/pronoun-meta-review-uk-nationals"

```

The generated PDF output looks like this:

![Simulchip PDF Output](/blog/img/simulchip_output.png)

## Looking Forward

Simulchip represents the intersection of practical problem-solving and thoughtful engineering. Having just gotten into the game myself, I want to make it as easy as possible for new players to join the community.

Several exciting developments are on the roadmap:

- **Textual Interface**: Migration to a textual-based curses-style application for a more interactive and visually appealing terminal experience
- **Online Hosting**: Plans to host the tool online, either as a web application or through a themed SSH endpoint that captures the runner-corp aesthetic
- **Private Proxying**: Simulchip currently requires the net deck to be public. However, I'm sure some people would want to prepare for tournaments without making their decklists public. We'll need an easy way of managing this.
- **Community-Driven Features**: Most importantly, whatever features the community wants—the direction will be shaped by actual users and their needs

The tool is available now at [github.com/dfiru/simulchip](https://github.com/dfiru/simulchip).

Whether you're a new player building your first collection or an experienced runner expanding into new archetypes, Simulchip handles the tedious parts so you can focus on what matters: the game itself.
