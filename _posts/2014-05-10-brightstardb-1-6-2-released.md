---
layout: post
title: BrightstarDB 1.6.2 Released
tags: release
---

I am pleased to announce release 1.6.2 of BrightstarDB is now available from all the usual places:

 * [MSI installer on CodePlex](https://brightstardb.codeplex.com/releases/view/121933 "BrightstarDB Installer Download")
 * [NuGet packages](https://www.nuget.org/ "NuGet.org")
 * [Documentation on ReadTheDocs.org](http://brightstardb.readthedocs.org/en/1.6.2/ "BrightstarDB Documentation")

This is a bug-fix release, fixing an issue with the database caching code which could cause corruption to database indexes. This is a recommended update for all users of BrightstarDB.

Thanks go to Phil Coppney for providing the bug report along with code to reproduce it.
