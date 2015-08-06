---
layout: post
title: BrightstarDB 1.11 Released
tags: release
---

I am pleased to announce release 1.11 of BrightstarDB is now available from all the usual places:

 * [MSI installer on CodePlex](https://brightstardb.codeplex.com/releases/view/616646 "BrightstarDB Installer Download")
 * [NuGet packages](https://www.nuget.org/ "NuGet.org")
 * [Documentation on ReadTheDocs.org](http://brightstardb.readthedocs.org/en/1.11/ "BrightstarDB Documentation")

This release adds a whole bunch of new features and improvements across the board, many of which are the direct result
of feedback and suggestions from the awesome BrightstarDB community of users (or perhaps that should be community of awesome users!).
Thanks to everyone who posted suggestions and reported bugs. You can get involved either on our [discussion list](https://brightstardb.codeplex.com/discussions)
or by opening an issue on our [GitHub project](https://github.com/BrightstarDB/BrightstarDB).

There is a breaking API change. The ExecuteQuery method on a BrightstarEntityContext now returns an ISparqlResult instance
rather than the raw Stream. The ISparqlResult interface provides accessors to determine if the result is an RDF graph or
a SPARQL result set and to access the graph or result set directly using the dotNetRDF IGraph and SparqlResultSet interfaces.
This change makes it much easier to work with SPARQL results. Please note that this change only affects code that makes direct
SPARQL queries - code that only uses LINQ will not be affected.

The store format remains compatible with previous releases. 

For full details you can read the [What's New](http://brightstardb.readthedocs.org/en/1.11/Whats_New) page of the documentation.
