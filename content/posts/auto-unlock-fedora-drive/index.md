---
title: "Auto Unlock Fedora Encrypted Drive"
description: "How to automatically unlock a LUKS encrypted drive with TPM2 in Fedora"
date: 2023-05-14T14:32:11+10:00
draft: false
categories:
  - Computing
tags:
  - tips
  - linux
  - future-reference
  - security
images:
  - "posts/auto-unlock-fedora-drive/password.jpg"
---
I recently installed Fedora on my personal laptop to replace Manjaro as I found Manjaro to be a just a little too flaky for my liking and I know Fedora to be a solid alternative. During installation I was asked if I wanted to encrypt my drives. Now I'm not harbouring anything particularly sensitive but I thought it was worth doing, security first and all that; after all, we expect every website to support HTTPS these days so why not?

Well there is the matter of that mildly pesky login prompt on every boot...

<!--more-->

Now, as any Linux user knows, for every problem there is a solution. It gets tricky when you try to find it and make it work for you! Fortunately Fedora Magazine already [put up a guide](https://fedoramagazine.org/automatically-decrypt-your-disk-using-tpm2/) on how to do it, and I'm going to basically distill what they've put in there and demonstrate how I made it work for me. Check out that page for a more in-depth explanation of what everything does.

## Preparation
1. Ensure Secure Boot is enabled:
    ~~~ bash
    $ dmesg | grep secureboot
    ~~~
    You should see an entry with something like 
    > `[    0.000000] secureboot: Secure boot enabled`

    If it shows as disabled, you'll need to enable it. That's out of scope for this post but you should be able to find a solution easily on The Internets. Just be aware that secure boot has the potential to mess with other software you have running (DisplayLink springs to mind, article to follow)

2. Ensure TPM2 chip is present:
    ~~~ bash
    $ dmesg | grep TPM2
    ~~~
    And again you'll see something like
    > `[    0.012155] ACPI: TPM2 0x0000000043BE1000 00004C (v04 INSYDE TGL-ULT  00000002 ACPI 00040000)`

    If you don't... might as well stop reading!

3. Install Clevis and necessary tools:
    ~~~ bash
    $ sudo dnf install clevis clevis-luks clevis-dracut clevis-udisks2 clevis-systemd
    ~~~

4. Prepare the system for binding the LUKS partition to TPM2 by regenerating initramfs:
    ~~~ bash
    $ sudo dracut -fv --regenerate-all
    ~~~

5. Reboot to ensure the latest changes are picked up by the system.

## Configuring Clevis
1. Execute the following command to bind the LUKS partition to the TPM2 module:
    ~~~ bash
    $ sudo clevis luks bind -d /dev/nvme0n1p3 tpm2 '{"pcr_ids":"1,4,5,7,9"}'
    ~~~

2. Automatic unlock may work at this point, try a restart and see. If not, the following trick should fix it as it did for me. It basically adds a delay before asking for a password to ensure that the required modules for unlocking are loaded.

    1. Edit the `systemd-ask-password-plymouth` service file:
        ~~~ bash
        $ systemctl edit systemd-ask-password-plymouth.service
        ~~~
    2. Add the following inside the file, taking note of where the comments say it should go:
        ~~~ ini
        [Service]
        ExecStartPre=/bin/sleep 10
        ~~~
    3. Open the file `/etc/dracut.conf.d/systemd-ask-password-plymouth.conf` for editing and add the following:
        ~~~ ini
        install_items+=" /etc/systemd/system/systemd-ask-password-plymouth.service.d/override.conf "
        ~~~
    4. Regenerate initramfs:
        ~~~ bash
        $ sudo dracut -fv ‐‐regenerate-all
        ~~~
    5. Reboot
    6. Regenerate the binding
        ~~~ bash
        $ sudo clevis luks regen -d /dev/nvme0n1p3 -s 1 tpm2
        ~~~

## Reconfiguring Clevis after system upgrade
When upgrading the kernel for example, initramfs is regenerated and that requires an update to the binding. Perhaps there's a way to trigger this when initramfs is updated but I'm not aware of one, so for now I have the following in a script to run when the password prompt makes an appearance again.

``` bash
!# /bin/zshs

sudo clevis luks regen -d /dev/nvme0n1p3 -s 1 tpm2
```