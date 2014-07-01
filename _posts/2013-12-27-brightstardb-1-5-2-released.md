---
layout: post
title: BrightstarDB 1.5.2 Released
tags: release
---

We are pleased to announce the release of version 1.5.2 of BrightstarDB. This version is primarily a bug-fix release correcting some issues found in the 1.5.1 release of BrightstarDB with the SPARQL query browser interface.

Release 1.5.1 was released without announcement here, but its major new features over 1.5 were:

    Added support for Visual Studio 2013 in the installer. The installer can now integrate the BrightstarDB item templates into Visual Studio 2013 Professional edition (or above). Regrettably this functionality is not available in the various Express editions of Visual Studio.
    Extended the SPARQL query API to support the specification of the serialized results format. This enables query results to be returned in a variety of formats including different RDF serializations for CONSTRUCT and DESCRIBE queries.
    Added a method for listing all of the jobs currently queued or recently completed for a store.

As usual the updated installer can be downloaded from our [CodePlex project page](https://brightstardb.codeplex.com/), and updated [NuGet](http://www.nuget.org/packages?q=brightstardb) packages are also available. The full documentation is also online at [ReadTheDocs.org](http://brightstardb.readthedocs.org/en/latest).
