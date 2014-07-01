---
layout: post
title: BrightstarDB 1.4 Alpha Released
tags: release prerelease pcl
---

I am pleased to announce that the first pre-release version of BrightstarDB 1.4 is now available for download via NuGet. The main advances in this release are the work on memory usage during import; and the new Portable Class Library build. I’ve written a bit about the memory usage (for example [here](/2013/07/15/update.html)) so in this post I just want to talk a bit about the Portable Class Library build.

So, why PCL ? Well, we already have a Windows Phone library that works nicely for Windows Phone 7.1 and above; but the biggest missing piece was of course Windows Store Apps. PCL allows us to not only target Store Apps but also Silverlight and Windows Phone all with a single assembly. I strongly believe that there is a place for a small, compact triple store as part of desktop and mobile applications and so being able to support developers of Windows Store Apps was an item quite high on my priority list. And also I just think that the experience of developing the the BrightstarDB entity framework is way nicer than the native storage options provided for Windows Store Apps.

You will see that the PCL API sticks very closely to the standard BrightstarDB API. I didn’t introduce a lot of asynchronous wrappers around the essentially synchronous operations of BrightstarDB. At some point I may introduce some wrappers to make it easier to await the completion of a transaction or SPARQL update, but for now you will have to do that yourself in your code. Equally you can always execute operations on a separate thread if you want to maintain UI responsiveness during long running queries for example.

You will also see that the set of platforms supported are restricted to the most recent versions (SL5, Windows Phone 8, .NET 4.5) the main reason for that was that differences in the core library meant it was not possible to support Entity Framework on the earlier versions, so in the end I opted for producing a single PCL that targets all platforms that allow us to provide the Entity Framework piece.

If you want to play around with the Portable Class Library, I recommend you [read the docs about using the Portable Class Library](http://brightstardb.readthedocs.org/en/develop/Developing_Portable_Apps/) and then go grab the NuGet packages (remember these are pre-release packages so you will need to configure your NuGet package manager or specify -IncludePrerelease on the command line). With so many new platforms added, I would really welcome any bug reports or thoughts on how we can make things better for app developers. If you have any comments please share them here or on the BrightstarDB Google group.