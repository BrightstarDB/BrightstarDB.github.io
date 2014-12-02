---
layout: post
title: BrightstarDB 1.8.0 Released
tags: release
---

I am pleased to announce release 1.8.0 of BrightstarDB is now available from all the usual places:

 * [MSI installer on CodePlex](http://brightstardb.codeplex.com/releases/view/158829 "BrightstarDB Installer Download")
 * [NuGet packages](https://www.nuget.org/ "NuGet.org")
 * [Documentation on ReadTheDocs.org](http://brightstardb.readthedocs.org/en/1.8/ "BrightstarDB Documentation")

This update fixes some bugs and addresses performance issues reported by the community. Thanks to all those who 
took the trouble to report and to provide patches / suggested workarounds.

Key new features in this release are:

 * EntityFramework now supports GUID properties.
 * EntityFramework now has an [Ignore] attribute which can be used to decorate interface properties that are not to be implemented by the generated EF class.
 * Added a constructor option to generated EF entity classes that allows property initialisation in the constructor.
 * Added some basic logging support for Android and iOS PCL builds. 
 * It is now possible to iterate the distinct predicates of a data object using the GetPropertyTypes method.
  
Significant fixes in this release are:
  
 * Fix for Polaris crash when attempting to process a query containing a syntax error.
 * Fixed NuGet packaging to remove an obsolete reference to Windows Phone 8. WP8 (and 8.1) are still both supported but as PCL profiles.
 * Performance fix for full cache scenarios.
           
The store format remains compatible with previous releases. This is a recommended update for all BrighstarDB users.

Docker Image Now Available 
--------------------------

With this release we are now also providing a Docker image to run a BrightstarDB server in a Docker container. This makes it really easy to get a BrightstarDB service up and running on a cloud VM infrastructure such as Azure or AWS. The docker image is [available on Docker Hub]("https://registry.hub.docker.com/u/brightstardb/brightstardb/" "BrightstarDB on Docker Hub"). For more information please read our notes in the 
[BrightstarDB/Docker repository](https://github.com/BrightstarDB/Docker) on GitHub where you will also find the Dockerfile and configuration files used to build the image.