---
title: "Await an Async GDScript Function From C#"
description: "How to await GDScript functions when called from C#... sometimes"
date: 2024-12-12T18:04:32+10:00
draft: false
categories:
  - Game Development
tags:
  - programming
  - game-development
  - godot
  - tips
  - future-reference
series:
  - Godot
---
I like that Godot has its own, simple scripting language in GDScript, but I'm a .NET tragic so it's C# for me. Interoperability between the two is possible, but not always obvious. Case in point, how do you await an async GDScript method? Well, it depends...
<!--more-->
I'm going to use the [Universal Fade](https://github.com/KoBeWi/Godot-Universal-Fade) addon as an example. Awaiting a fade-in with GDScript could hardly be simpler.

``` gdscript
await Fade.fade_in().finished
```

Calling GDScript from C# is a lot more involved and suffers from inflexibility.

``` csharp
Script fadeScript = ResourceLoader.Load("res://addons/UniversalFade/Fade.gd") as Script;
fadeScript.Call("fade_in");
```

It works, but it's not async. If you want to continue from the point the fade has finished, you need to latch on to the `finished` signal that is thankfully provided by the addon. This is the "it depends" part, it depends on how the developer has configured the script.

``` csharp
Script fadeScript = ResourceLoader.Load("res://addons/UniversalFade/Fade.gd") as Script;
var fader = fadeScript.Call("fade_in");
await fadeScript.ToSignal(fader.AsGodotObject(), "finished");
```

Wonderful! And with this knowledge I put together a static helper class to fade in and out, asynchronously with awaiters.

``` csharp
public class FaderHelpers
{
    public static async Task FadeOut(float duration)
    {
        Script fadeScript = ResourceLoader.Load("res://addons/UniversalFade/Fade.gd") as Script;
        var fader = fadeScript.Call("fade_out", duration);
        await fadeScript.ToSignal(fader.AsGodotObject(), "finished");
    }

    public static async Task FadeIn(float duration)
    {
        Script fadeScript = ResourceLoader.Load("res://addons/UniversalFade/Fade.gd") as Script;
        var fader = fadeScript.Call("fade_in", duration);
        await fadeScript.ToSignal(fader.AsGodotObject(), "finished");
    }
}
```

Et voila! Simply call with the duration of the fade.

``` csharp
await FaderHelpers.FadeOut(5);
```