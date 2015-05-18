---
layout: post
title: BrightstarDB 1.10.1 Released
tags: release
---

We have just pushed out a 1.10.1 hotfix release for the Windows installer of BrightstarDB.

This release fixes an issue with the BrightstarDB server failing to render HTML pages when accessed
via the browser. This problem was caused by a missing System.Web.Razor.dll and would only be encountered
on machines where ASP.NET MVC had not been previously installed. The installer has now been updated
to include this required DLL.

Many thanks to Martin Lercher for the bug report!

You can [download the updated installer from CodePlex](https://brightstardb.codeplex.com/releases/view/615137 "BrightstarDB Installer Download")

Please note, we are not updating the NuGet packages at this time as this hotfix is only a fix for a 
packaging issue with the installer.
