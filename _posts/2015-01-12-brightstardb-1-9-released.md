---
layout: post
title: BrightstarDB 1.9.0 Released
tags: release
---

I am pleased to announce release 1.9.0 of BrightstarDB is now available from all the usual places:

 * [MSI installer on CodePlex](http://brightstardb.codeplex.com/releases/view/599056 "BrightstarDB Installer Download")
 * [NuGet packages](https://www.nuget.org/ "NuGet.org")
 * [Documentation on ReadTheDocs.org](http://brightstardb.readthedocs.org/en/1.9/ "BrightstarDB Documentation")

This update adds some new features that have been requested and proposed by the user community. Thanks to all those who took the trouble to report and to provide patches / suggested workarounds.

Key new features in this release are:

 * EntityFramework now supports using an empty string as the Identifier prefix for an entity.
   This allows you to create EF entities with the full resource IRI specified in the Id property.
 * The Polaris UI now allows the default graph IRI to be specified for import operations.
 * The REST API now reports error messages back to the client along with the 400 status code.
 * It is not possible to disable transaction logging by deleting the transactionheaders.bs
   file for a store (you can then also delete transactions.bs to save space). It is also
   now possible to specify whether an embedded BrightstarDB instance defaults to having
   a transaction log created for new stores. By default transaction logging is disabled
   for mobile platforms and enabled for other platforms.

  
The store format remains compatible with previous releases. There is a minor API change 
which may require code to be updated if it was using the BrightstarDB.Configuration API.

This is a recommended update for all BrighstarDB users.

The Docker image will be updated shortly.