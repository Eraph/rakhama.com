---
title: "The Mystery of the Atari ST's Bad DMA: Solved?"
description: "More analysis on the infamous Atari ST Bad DMA hard drive problem"
date: 2023-05-10T19:14:39+10:00
draft: false
categories:
  - Retro Computing
tags:
  - atari st
  - retro computing
  - linked article
series:
  - Atari
images:
  - "68000.jpg"
---

When I set up my own SD-HDD solution based on the wonderful [ACSI2STM](https://github.com/retro16/acsi2stm) project, I found the hard drive was becoming corrupted pretty soon after setting it up. I thought it might have been my dodgy soldering or something funny about the way I had everything set up. Turns out I simply hadn't heard about [Bad DMA](https://exxosforum.co.uk/articles/DMA.html). A local Atari enthusiast meetup later and a solution was proposed; swap the 68000 chip for a newer, lower power and subsequently less noisy version.

It worked!

<!--more-->

At least, I haven't had any corruption happening since the swap. But a [recent article](https://www.chzsoft.de/site/hardware/new-atari-ste-bad-dma-investigation/) from CHZ-Soft follows a deep dive into investigating the root cause of the problem. The verdict seems to be that I'm one of the lucky ones. The problem is teetering on a knife edge, some unknown combination of variables just happened to allow my system to work with a new processor. A more robust solution would be to patch EmuTOS (but since I'm lazy and everything works, I'll just hope they fold it in to the codebase) as explained near the end of the article.

Read the full article here: [A new Atari STE bad DMA investigation](https://www.chzsoft.de/site/hardware/new-atari-ste-bad-dma-investigation/).