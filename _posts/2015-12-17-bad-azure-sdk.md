---
layout: post
title: BrightstarDB Build / Azure SDK v2.8 Incompatibility ?
tags: how-to
---

This should not be an issue for you to worry about unless you are trying to build BrightstarDB from source.

TL;DR: If you are getting wierd compiler errors on BrightstarDB.csproj that say that dotNetRDF is pulling in
a Newtonsoft.JSON.dll dependency that targets .NET 4.5; then try uninstalling Azure SDK v2.8 from your machine.

However, I recently came across a problem building BrightstarDB from source on just one machine out of three,
the error shows up as load of compile time errors when building BrightstarDB.csproj which all have as their
root cause a compiler *warning* about not being able to resolve a reference to dotNetRDF.dll. The warning
says that this is because it in turn has a dependency on Newtonsoft.JSON.dll 6.0.0.0 and that the DLL it
found for *that* reference is targeting .NET 4.5. This causes the compiler to give up resolving the
dependency because the BrightstarDB.csproj project targets a lower version of the .NET framework (4.0).

After much repairing and reinstalling of VS2015; clearing the package caches for NuGet and so on I finally
tracked down the problem by looking at the few generated files that had been put in the obj\Debug directory
of the BrightstarDB project. One of those is BrightstarDB.csproj.ResolveAssemblyReference.cache. It is a 
binary file, but Notepad++ was happy enough to open it and it does contain plenty of readable strings. Searching
it for Newtonsoft.JSON.dll; I found that the resolved reference was actually going to a copy of the assembly in
a directory under  ``C:\Program Files\Microsoft SDKs\Azure\.NET SDK\v2.8\bin\plugins``, in preference to using
the assembly contained in the Newtonsoft.JSON nuget package that BrightstarDB.csproj references.

I have *absolutely no idea* why the C# compiler should want to look in this Azure SDK *plugin* directory. There
is nothing in the csproj that references that location or indeed the Azure SDK.

Anyway, the fix I used was simple: Uninstall Azure SDK v2.8

Along the way I've learned more than I wanted to about NuGet package caching (there are *two* separate full
pacakage caches, ``nuget locals -list all`` will show you where they are); just how long it takes to reinstall
VS2015; and the gruesome internals of assembly dependency resolution in the C# compiler. Hopefully this post
might save someone else a similar "voyage of discovery"!