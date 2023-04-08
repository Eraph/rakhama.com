---
title: "USB Floppy Emulator - Gotek"
description: "How to install and configure a Gotek USB Floppy Emulator for the Atari ST."
date: 2023-04-02T08:21:59+10:00
draft: true
categories:
  - Retro Computing
tags:
  - atari-st
  - tips
  - retro-computing
  - hardware
series:
  - Atari
images:
  - "path/to/something.jpg"
---
A quick guide on using a Gotek USB floppy emulator in your ST, conveniently store all your floppy images on a single drive and never have to worry about worn out floppy drive belts again!

<!--more-->

The Gotek was the first accessory I got for my ST. It makes sense after all, the floppy drive is the only way to interact with data on your ST by default. It's well worth having even if you intend to set up an HDD as many games struggle with being run from a hard drive, and many more don't work well with EmuTOS, which [I recommend](/posts/tos-is-rubbish) as a modern replacement to the installed operating system.

## Getting a Gotek
I purchased my Gotek through Ebay from a local seller, there doesn't seem to be an "official" place to purchase them from but searching for `Gotek USB` will give you several places to purchase them from. The Gotek I use does not have a rotary encoder which I don't believe will fit in the ST's case without modification.

## Installing the Gotek
- Twisted cable?

## Using the Gotek
With only two buttons and a 3-digit numeric display, there's not much to it! The right button increments the counter through the assigned slots, and the the left button predictably decrements the counter. Pressing both resets the counter to slot 0, which is one of two reserved slots:

- **Slot 0**: FlashFloppy configuration
- **Slot 1**: Selecting a disk image from FlashFloppy with the F7 key puts the image in this slot and immediately boots

## Setting up FlashFloppy
[FlashFloppy](https://github.com/keirf/flashfloppy) is an open-source floppy disk emulator firmware for [Gotek drives](https://github.com/keirf/flashfloppy/wiki/Gotek-Models). It includes functionality to configure the Gotek and map floppy images to slots in the Gotek.
- Intro to FlashFloppy
  - Setting up pages
  - Keeping track with ExCeL mofo