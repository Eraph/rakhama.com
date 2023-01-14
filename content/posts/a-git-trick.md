---
title: "Post #1: A Git Trick"
description: "Configure Git to always create a remote branch on push if it doesn't already exist."
date: 2021-11-02T19:37:16+10:00
draft: false
categories: 
  - Programming
tags:
  - programming
  - git
  - tips
  - future reference
---
Hello! I've started a new blog. For my first post I'm going to share a Git configuration trick that I often use to work around this dang ol' error message when trying to push a locally created branch to a remote repository:

<!--more-->

> fatal: The current branch &lt;X&gt; has no upstream branch.

Set Git to always create the remote branch on push:

``` bash
git config --global push.default current
```