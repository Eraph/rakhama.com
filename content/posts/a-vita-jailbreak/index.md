---
title: "A Vita Jailbreak"
description: "On unlocking the PlayStation Vita's full potential"
date: 2025-10-19T01:08:36Z
draft: true
categories:
  - Retro Computing
tags:
  - tips
  - retro-computing
  - retro-gaming
  - ps-vita
  - gaming
images:
  - "mainImage.jpg"
---
The PS Vita is a mighty little handheld that stands up very well in 2025 (except for its WiFi, it's grim). I got mine well over ten years ago and jailbroke it a couple of years ago. It has been throwing a few errors with certain things, in particular the infamous `C2-12828-1` error, I thought maybe it needs a big ol' refresh. So this post is about backing up savegames, resetting everything, and then setting it up again from scratch.
<!--more-->
## Basics
I use [VitaShell](https://www.cfwaifu.com/vitashell/) to manage file content on the Vita, and it has a very handy FTP feature (turn it on in the app settings by changing the function of the `SELECT` button). USB mode will also work if you're more comfortable with that.

## Backing Up
There wasn't much I wanted to backup aside from saves. There were two main kinds of saves I was interested in; Vita saves, and PS1/PSP saves (via Adrenaline, the PSP emulator). The latter are easily backed up by copying the contents of `ux0:/pspemu/PSP/SAVEDATA` to your computer. For the former, there is the [Vita Save Manager Plus](https://www.gamebrew.org/wiki/Vita_Save_Manager_Plus) app. Select the games with saves you want to backup, and copy the folders from `ux0:/data/savegames` to your computer.

## Factory Reset
Once you're ready to clear everything out, turn off the Vita. Boot it into the recovery menu by holding down `R` + `Power` + `PS`. In a couple of seconds the PlayStation logo will appear, and the buttons can be released. Select `Restore This System` and follow the prompts. Note that this will clear out everything on the Vita, including custom firmware. It's not necessary to format the memory card yet, We'll get to that later.

## Jailbreak
Time to unleash the Vita's full potential! Jailbreaking the device couldn't be more straight-forward, and the benefits are endless. [Vita Hacks Guide](https://vita.hacks.guide/) is an excellent resource for getting started, and the one I followed. Follow the process outlined there and you'll have an unlocked Vita in no time.

From here, make sure you have the following installed:
- iTLS
- VitaDeploy
- VitaShell
- jPKG

## Log in to PSN
Before logging in to PSN, check the appropriate settings are in place by going to `Settings` > `HENkaku Settings`. Ensure `Enable PSN Spoofing` and `Enable Version Spoofing` are checked.

Logging in to PSN will ask if you want to log in as the user connected to the memory card. I wanted a completely fresh start, so I said "no", prompting the card to be formatted. Then, as if to spite the system for even asking, I logged in as the same user anyway.

## Install Apps
[VitaDB Downloader](https://www.rinnegatamante.eu/vitadb/#/info/877) is the app to use for downloading homebrew apps to your Vita. It should have been installed when jailbreaking. Here are some apps I recommend:

- [AutoPlugin2](https://github.com/ONElua/AutoPlugin2) - install and manage plugins with ease
- [Adrenaline](https://www.rinnegatamante.eu/vitadb/#/info/47) - allows access to the embedded PSP emulator; ideal for PSP and PS1 games
- [RetroArch Nightly](https://www.rinnegatamante.eu/vitadb/#/info/186) - frontend for other emulators; Mega Drive, SNES, loads more. Warning: very big download
- [Vitaki](https://www.rinnegatamante.eu/vitadb/#/info/1293) - PS5 remote play. Warning: latency is poor, I still live in hope that they may fix it
- [Stock App Remove](https://www.rinnegatamante.eu/vitadb/#/info/764) - unclutter your home screens by removing unwanted bubbles
- [vita-savemgr](https://www.rinnegatamante.eu/vitadb/#/info/9) - to restore those saves from earlier

## Install Plugins

## Install Games

## Restore Saves