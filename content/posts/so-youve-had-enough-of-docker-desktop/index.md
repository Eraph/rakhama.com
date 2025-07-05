---
title: "So You've Had Enough of Docker Desktop"
description: "Docker Desktop isn't the only way to manage your containers. Introducing Podman!"
date: 2024-10-22T08:49:49+10:00
draft: true
categories:
  - Computing
tags:
  - tips
  - linux
  - windows
  - open-source
  - podman
  - opensuse
images:
  - "podman.png"
---
Ah, the ol' Switch n' Bait. Please, use our free product, enjoy, have fun, okay now give us all your dollaridoos if you want to keep using it.
Easily one of my least favourite of all the terrible corporate money-making strategies.
As consumers we have the freedom (and in my opinion, responsibility) to look around for alternatives.
Fortunately for us, Docker is based on a standard set out by the [Open Container Initiative](https://opencontainers.org/) (somewhat ironically, Docker's own doing).

In this article I will introduce [Podman](https://podman.io/) as an alternative, and demonstrate how I manage my Podman setup.
<!--more-->

## Say Hello to Podman

> Podman is an open source container, pod, and container image management engine. Podman makes it easy to find, run, build, and share containers.

Yep, Podman is a free, open source, OCI compatible tool that is for most intents and purposes, drop-in compatible with Docker.
It's daemonless, meaning containers don't need to run as root making it inherently more secure and easier for end users to work with.
Containers spun up as part of a Compose command are grouped into pods.
And while there is a Podman desktop app, although I've had limited success with it in the past, hence this write-up.

## Preparing WSL

I'll be installing the Podman engine onto WSL.
First we need to configure WSL. Using Notepad or your text editor of choice, open or create `%UserProfile%\.wslconfig` (e.g. `C:\Users\phil\.wslconfig`)

``` ini
[wsl2]
vmIdleTimeout=-1

[experimental]
networkingMode=mirrored
```

The two options in here will ensure the running WSL instance doesn't time out and shares the network interface of the host machine.
If you have WSL running already, shut it down for the new settings to take effect on next startup.

``` PowerShell
wsl --shutdown
```

If you don't have WSL installed, use the following command to see what distributions are available.

``` PowerShell
wsl -l -o
```

I'm going to go with openSUSE to keep things interesting, but feel free to choose your own distribution (Ubuntu is a good option if you're new to Linux, and the default if you don't specify a distribution).
To install:

``` PowerShell
wsl --install -d openSUSE-Tumbleweed
```

Now start WSL by opening it from Windows Terminal (there should be a new entry for it) or by running it:

``` PowerShell
wsl
```

## Get Installin'

Install the following packages; I'm assuming whatever distribution you chose has them present.

- podman
- python310-podman-compose (cross-check this with the version of Python you have and check your repos for an appropriate version if necessary)
- cockpit
- cockpit-podman
- podman-docker (optional, allows you to use the `docker` command to run `podman`, useful if you're working with existing scripts)

``` bash
sudo zypper in podman python310-podman-compose cockpit cockpit-podman podman-docker
```

Now we need to make sure Cockpit is set to run as a service

``` bash
sudo systemctl enable cockpit.socket
sudo systemctl start cockpit.socket
```

## Working with Podman

Now if all went well you should be able to navigate to [https://localhost:9090](https://localhost:9090) and see a login screen for Cockpit. Login with your WSL credentials and feel the power flowing through your veins.

![The Podman plugin running in Cockpit](/posts/so-youve-had-enough-of-docker-desktop/cockpit-podman.png)

I won't go into heaps of detail about working with Cockpit, it's simply a nice interface for working with containers (and your WSL instance) if you need it.

As for working with Podman, well the commands are practically identical to Docker. Substitute the word `docker` for `podman` (or don't, if you installed `podman-docker`), and you're 99% of the way there.
It also has support for compose files if you have those, for example:

``` bash
podman compose -f docker-compose.yml up -d
```

## Auto-Starting Containers

Of course, Podman isn't perfect. I'm sure there are edge cases where it doesn't quite align with how Docker does things. One thing that stands out is that daemonless (i.e. running under the context of the logged-in user) containers don't start on boot even when the restart policy is set in such a way that they would with Docker - at least, not without a fight. There are workarounds but they all feel quite heavy handed in my opinion. There is another option though, don't run the container under the user context.

First, ensure the `podman-restart` service is enabled

``` sh
sudo systemctl enable podman-restart
```

Now when creating the container, prefix with `sudo` to run the container in a "rootful" manner, and specify the restart policy:

``` sh
sudo podman run -dt -p 8080:80/tcp --restart=always docker.io/library/httpd
```

This should also be applicable to `podman compose` and other commands. Next time the host reboots, the container should start up automatically.