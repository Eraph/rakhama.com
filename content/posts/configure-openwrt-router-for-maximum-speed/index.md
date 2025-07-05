---
title: "Configure OpenWRT Router for Maximum Speed"
description: "About this page"
date: 2025-01-04T08:32:57+10:00
draft: true
categories:
  - One
tags:
  - One
series:
  - One
images:
  - "path/to/something.jpg"
---
Given an internet connection with imposed upload/download limits, a router may attempt to exceed those limits, kicking in some kind of overzealous throttling that can negatively affect latency. The trick is to set maximum limits on the router itself, just a couple of Mbit short of your plan's limits. This article explains how to set this up for the GL.inet Flint 2 router, but should be applicable to any router with OpenWRT.
<!--more-->
- Install `luci-app-qos`
- Set up in LUCI console
- Speed tests from router?