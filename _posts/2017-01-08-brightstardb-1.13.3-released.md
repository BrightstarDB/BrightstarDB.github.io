---
layout: post
title: BrightstarDB 1.13.3 Released
tags: release
---

A new hotfix for BrightstarDB 1.13 has just been released. This release fixes an issue with the stability of the hashcode of EntityFramework instances that
caused issues for libraries using the observable interface of EF collections. This is a recommended update for all BrightstarDB users.

The new release is now available from all the usual places:

 * [Release on GitHub](https://github.com/BrightstarDB/BrightstarDB/releases/tag/1.13.3 "BrightstarDB Installer Download")
 * [NuGet packages](https://www.nuget.org/ "NuGet.org")
 * [Documentation on ReadTheDocs.org](http://brightstardb.readthedocs.org/en/1.13/ "BrightstarDB Documentation")

Thanks (again!) to GitHub user osc117 for the bug report and useful reproduction project. 
