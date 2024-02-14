---
title: "Scoop Upgrade"
description: "Scoop is great, use Scoop. Also, add this alias for great justice."
date: 2024-02-14T13:29:17+10:00
draft: false
categories:
  - Software
tags:
  - tips
  - future-reference
  - windows
  - open-source
  - alternatives
images:
  - "posts/scoop-upgrade/scoopupgrade.jpg"
---
Are you still downloading `.exe` files like some kinda chump? Check out [Scoop](https://scoop.sh), a package manager for Windows not too dissimilar to [Chocolatey](https://chocolatey.org) which ought to be very familiar to any Linux enthusiast. It's fast, it's easy to use, and it's open source. Yes, Scoop is great but this article is not about extolling the virtues of the package management tool.

<!--more-->
You see, out of the box it's missing a pretty key feature! Users of `apt-get` will be familiar with the `upgrade` argument which, well, upgrades all upgradeable installed packages. It is possible to pass in an asterisk as a parameter to the `scoop update` command, but for simplicity I'd like a dedicated command. Well fam, Scoop got ya back. Check out the help text for the `scoop alias` command:

``` bash
Usage: scoop alias add|list|rm [<args>]

Add, remove or list Scoop aliases

Aliases are custom Scoop subcommands that can be created to make common tasks
easier.

To add an Alias:
    scoop alias add <name> <command> <description>

e.g.:
    scoop alias add rm 'scoop uninstall $args[0]' 'Uninstalls an app'
    scoop alias add upgrade 'scoop update *' 'Updates all apps, just like brew or apt'

Options:
  -v, --verbose   Show alias description and table headers (works only for 'list')
```

It only goes and tells you how to add an alias for updating all the apps:

``` bash
scoop alias add upgrade 'scoop update *' 'Updates all apps, just like brew or apt'
```

Now next time you want to update all installed apps, just hack it in:

``` bash
scoop upgrade
```