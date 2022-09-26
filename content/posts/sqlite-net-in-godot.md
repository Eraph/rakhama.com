---
title: "SQLite.NET in Godot"
date: 2022-07-26T20:02:53+10:00
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
I'll talk more about why I think the [Godot game engine](https://godotengine.org/) is a good alternative to [Unity](https://unity.com/) as part of my alternatives series of posts, but for now I want to share for posterity's sake how to use SQLite.NET in Godot. Skip to the end to add the one package necessary to get SQLite.NET working in Godot.

For the uninitiated, [SQLite.NET](https://github.com/praeclarum/sqlite-net) is an ORM for SQLite, similar to Entity Framework. SQLite is a good fit for games and I'm fond of using an ORM where possible. The PCL version is cross-platform, so it'll run on Android and all sorts.

As of now the latest version of [sqlite-net-pcl](https://www.nuget.org/packages/sqlite-net-pcl) on Nuget is 1.8.116, but this seems to be incompatible with Godot or .NET 4.7.2, resulting in the following error:

> Library e_sqlite3 not found

I've spent an awful lot of time trying to tease out the solution, referencing explicit SQLite3 nuget packages (so many of them,) mixing and matching combinations of the aforementioned, creating my own GDNativeLibrary to explicitly include them with the project... all with the same result.

The trick is, as alluded to in [this tucked away post](https://github.com/praeclarum/sqlite-net/issues/967#issuecomment-653799871), to use version 1.6.292.

```bash
dotnet add package sqlite-net-pcl --version 1.6.292
```

And that's it! No other packages necessary.
