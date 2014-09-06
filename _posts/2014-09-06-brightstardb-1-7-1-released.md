---
layout: post
title: BrightstarDB 1.7.1 Released
tags: release
---

I am pleased to announce release 1.7.0 of BrightstarDB is now available from all the usual places:

 * [MSI installer on CodePlex](https://brightstardb.codeplex.com/releases/view/132297 "BrightstarDB Installer Download")
 * [NuGet packages](https://www.nuget.org/ "NuGet.org")
 * [Documentation on ReadTheDocs.org](http://brightstardb.readthedocs.org/en/1.7.1/ "BrightstarDB Documentation")

This update fixes a couple of bugs and also adds MonoTouch portable class libraries for developing iOS apps with
BrightstarDB. 

 * NEW: iOS is now supported in the Portable Class Library build.
 * FIX: Fixed SPARQL endpoint to properly return the requested format when it is specified using the format= query parameter.
 * FIX: Fixed build of BrightstarDB on Mono. Thanks to abargnesi for the bug report.
 * FIX: Fixed serialization of decimal, double and float data-types to ensure use of the Invariant culture. Thanks to CyborgDE for the bug report.
 * UPDATE: All projects are now being built with VS2013.
 
 The store format remains compatible with previous releases. This is a recommended update for all BrighstarDB users.