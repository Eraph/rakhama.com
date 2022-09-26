---
title: "Slight Delay With Bluetooth Mouse in Linux"
date: 2022-08-18T21:11:26+10:00
draft: false
categories:
  - Computing
tags:
  - tips
  - linux
  - future-reference
---

A quick tip here for when a bluetooth mouse takes a moment to react after not being used for a few seconds. Bluetooth autosuspend can be disabled with a negligible power consumption penalty by setting a kernel parameter on boot.

<!--more-->

1) Open `/etc/default/grub` for editing as root.
2) Find the line with `GRUB_CMDLINE_LINUX_DEFAULT` and append `btusb.enable_autosuspend=0` to the end.
3) Save your changes.
4) Run:
    ``` bash
    sudo update-grub
    ```
5) Reboot.

This solution was found [here](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1801642) and is confirmed as a fix for Manjaro.