---
layout: post
title: BrightstarDB 1.3 Released
tags: release
---
We are pleased to announce the release of BrightstarDB 1.3. This is the first “official” release of BrightstarDB under the [open-source MIT license](http://opensource.org/licenses/MIT). All of the documentation and notices on the website should now have been updated to remove any mention of commercial licensing. To be clear: BrightstarDB is not dual licensed, the MIT license applies to all uses of BrightstarDB, commercial or non-commercial. If you spot something we missed in the docs that might indicate otherwise please let us know.

The main focus of this release has been to tidy up the licensing and use of third-party closed-source applications in the build process, but we also took the opportunity to extend the core RDF APIs to provide better support for named graphs within BrightstarDB stores. This release also incorporates the most recent version of dotNetRDF providing us with updated Turtle parsing and improved SPARQL query performance over the previous release.

So what are you waiting for – go get it! The binaries including a the server executable and desktop GUI management application can be downloaded as an installer package from [our CodePlex project page](https://brightstardb.codeplex.com/releases/view/107983). You can also use NuGet to install the BrightstarDB or BrightstarDBLibs package into your solution. Documentation is available online via [ReadTheDocs](http://brightstardb.readthedocs.org/).