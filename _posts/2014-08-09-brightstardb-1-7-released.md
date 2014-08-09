---
layout: post
title: BrightstarDB 1.7.0 Released
tags: release
---

I am pleased to announce release 1.7.0 of BrightstarDB is now available from all the usual places:

 * [MSI installer on CodePlex](https://brightstardb.codeplex.com/releases/view/127210 "BrightstarDB Installer Download")
 * [NuGet packages](https://www.nuget.org/ "NuGet.org")
 * [Documentation on ReadTheDocs.org](http://brightstardb.readthedocs.org/en/1.7/ "BrightstarDB Documentation")

This is a significant update for BrightstarDB including a number of critical bug fixes and also benefits from
upgrading to the latest version of DotNetRDF. There are no changes to the store format, so it should be a 
drop-in replacement for 1.x BrightstarDB installations...EXCEPT...

**WARNING**: This release removes support for Windows Phone 7 and Windows Phone 7.1 and merges all mobile support
into a single PCL build. This PCL build targets Profile 158 (portable-net45+wp80+sl5+win8+MonoAndroid10+MonoTouch10)
the necessary platform specific shims are included for .NET 4.5, WP80, SL5, Win8 and MonoAndroid. MonoTouch support
will be available soon.