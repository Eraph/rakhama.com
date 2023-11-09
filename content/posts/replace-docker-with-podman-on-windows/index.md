---
title: "Replace Docker With Podman on Windows"
description: "How to install and alias Podman for use as a drop-in Docker replacement on Windows"
date: 2023-05-15T15:02:42+10:00
draft: true
categories:
  - Computing
tags:
  - tips
  - future reference
  - open source
images:
  - "posts/replace-docker-with-podman-on-windows/podman.jpg"
---
I'm sure many were caught out by the switch-and-bait carried out by Docker when they started charging users for Docker Desktop. Well there is an alternative in [Podman](https://podman.io/), an open source Docker alternative that adheres to the same OCI (Open Container Initiative) standards, meaning it's pretty much a drop-in replacement. 

<!--more-->

Full installation instructions can be found [here](https://github.com/containers/podman/blob/main/docs/tutorials/podman-for-windows.md), but I've described the important bits below.

## Setting up Podman
1. Remove Docker if you already have it installed. You don't need it any more.
2. Go to the Podman Releases page on Github and find the latest installer, it's under the **Assets** section near the bottom of a release. Download the `.msi` file.
    ![posts/replace-docker-with-podman-on-windows/assets.jpg]
3. Run the MSI to install the Podman client.
4. Open PowerShell and run the following to make sure it's installed properly:
    ~~~ sh
    > podman help
    ~~~
5. If you're all good, the next step is to initialise the WSL machine that runs the containers:
    ~~~ sh
    > podman machine init
    ~~~
6. Then you can start up the machine:
    ~~~ sh
    > podman machine start
    ~~~


2. Install [Podman Desktop](https://podman-desktop.io/downloads), you can find it on Chocolatey:
    ~~~ ps
    > choco install podman-desktop
    ~~~
3. Open your Powershell Profile, its location can be found within PowerShell:
    ~~~ ps
    > $PROFILE
    ~~~
    > `C:\Users\phil\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1`
4. Add the following line:
    ~~~ ps
    New-Alias docker podman
    ~~~
5. Open a new PowerShell window and confirm the alias works:
    ~~~ ps
    > docker --help
    ~~~

And there you have it, being an open source application you're no longer subject to the whims of shareholders. As a bonus this also plays nice with such as [Visual Studio Code's Docker extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker) to easily manage your running containers.