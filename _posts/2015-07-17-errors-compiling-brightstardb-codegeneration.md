---
layout: post
title: Errors compiling BrightstarDB.CodeGeneration
tags: how-to tips compiling
---

When attempting to compile BrightstarDB from the source currently on the develop branch you may run into problems
compiling the BrightstarDB.CodeGeneration project where the compiler reports the following message:

    The type 'System.Object' is defined in an assembly that is not referenced. You must add a reference to assembly 'System.Runtime, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a'.
    
Don't worry! The fix for this is simple. You just need to install the updated version of the .NET 4.5.2 Developer Pack.

There is a description of the problem in this [Microsoft Knowledge Base article](https://support.microsoft.com/en-us/kb/2971005)
and download links are provided in [KB article 2901951](https://support.microsoft.com/en-us/kb/2901951).
