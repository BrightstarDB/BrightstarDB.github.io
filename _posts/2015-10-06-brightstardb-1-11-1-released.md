---
layout: post
title: BrightstarDB 1.11.1 Released
tags: release
---

I am pleased to announce release 1.11.1 of BrightstarDB is now available from all the usual places:

 * [MSI installer on CodePlex](https://brightstardb.codeplex.com/releases/view/617774 "BrightstarDB Installer Download")
 * [NuGet packages](https://www.nuget.org/ "NuGet.org")
 * [Documentation on ReadTheDocs.org](http://brightstardb.readthedocs.org/en/1.11.1/ "BrightstarDB Documentation")

As the version number suggests, this is a bug-fix release and includes a couple of critical bug fixes. One bug fix
addresses and issue with the BrightstarDB Entity Framework not correctly escaping strings in generated SPARQL queries - 
leading to invalid SPARQL queries and the resulting failure of EF LINQ queries. The other critical fix addresses a 
concurrency issue in the BrightstarDB database file storage - this has the potential to corrupt the content of the 
cache (requiring a restart to clear the error) or in some circumstances the content of the database itself. Due to 
the nature of this issue, I therefore STRONGLY recommend that all users should upgrade to this release as soon as possible.

There are no API changes over 1.11  and the store format remains compatible with previous releases.

For full details you can read the [What's New](http://brightstardb.readthedocs.org/en/1.11.1/Whats_New) page of the documentation.
