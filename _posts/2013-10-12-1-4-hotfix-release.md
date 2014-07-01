---
layout: post
title: 1.4 Hotfix Release
tags: release
---

Just a quick post to let you know that I pushed out a hotfix release for BrightstarDB 1.4 today. This release fixes a problem with .NET 4.0 applications using the WCF client. The problem occurred because the generated WCF client code (generated under .NET 4.5) included async methods returning Task<T> – unfortunately the .NET 4.0 version of the WCF framework knows nothing about these and complains that the return type is not serializable and throws an error at initialization time, even if the code in question is not using the async methods.

The update simply excludes the async methods from the generated WCF client code as these were never being used by BrightstarDB anyway. The update version number is 1.4.1011.0

You can grab an updated installer from our [Codeplex page](http://brightstardb.codeplex.com/). If you used NuGet, updated packages are now available via that route too.