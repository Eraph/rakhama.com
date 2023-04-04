---
title: "Debug Server Hosted Blazor Web Assembly in VS Code"
description: "Setting up server hosted Blazor WASM debugging in VS Code the easy way!"
date: 2023-04-04T16:20:25+10:00
draft: false
categories:
  - Programming
tags:
  - blazor
  - future-reference
  - programming
  - .net
  - tips
---
A published Blazor WebAssembly instance is just a collection of files that can be loaded into a browser. If these files are hosted on the API, running the API means the UI is running as well, probably from the same address. In this case it's a little different to debug compared to launching it standalone, but not any more complicated.

<!--more-->

## Prerequisites
You will need the following:
* Chrome or Edge, no other browsers are supported, even other Chromium-based browsers such as Chromium or Brave.
* The [C# Extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) for VS Code.

## Setting up

In our case, the API runs on a URL like `https://localhost:2200/api` and the Blazor WASM UI can be accessed from `https://localhost:2200`. To debug the UI, another debug session can be spun up in VS Code. To add a new launch profile, press `Ctrl+Shift+P` and select `Debug: Add Configuration...` or edit `launch.json` directly. Add the following configuration:

``` json
{
    "name": "Attach to Hosted UI",
    "type": "blazorwasm",
    "request": "attach",
    "url": "https://localhost:2200"
}
```

To launch and debug both the API and the UI at the same time, add the following (where `Debug API` is the name of the API launch task) as a [Compound Configuration](https://code.visualstudio.com/Docs/editor/debugging#_compound-launch-configurations):

``` json
{
    "name": "Debug Full-Stack",
    "configurations": [
        "Debug API",
        "Attach to Hosted UI"
    ],
    "stopAll": true
}
```

It's worth noting that [Microsoft's Blazor Debugging Documentation](https://learn.microsoft.com/en-us/aspnet/core/blazor/debug?view=aspnetcore-6.0&tabs=visual-studio-code#debug-hosted-blazor-webassembly-1) suggests the need to add special tags to the `.csproj` file. This isn't necessary for local debugging at least, but may help if you need to debug a published app.