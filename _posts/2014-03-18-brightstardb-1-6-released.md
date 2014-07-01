---
layout: post
title: BrightstarDB 1.6 Released
tags: Release
---
We are please to announce release 1.6 of BrightstarDB is now available. As usual you can get the packages via [NuGet](http://www.nuget.org/packages?q=brightstardb) or download the installer from our [Codeplex project page](http://brightstardb.codeplex.com/).

There are a bunch of new things added in this release covering a number of different areas.

Android Support
---------------

For mobile developers we have added support for Android via Xamarin.Android. This gives you the ability to run a BrightstarDB triple store on an Android device and use the BrightstarDB entity framework in your Xamarin.Andorid application. This is an initial experimental release to get some feedback so that we can work on improving both the developer and the user experience when using BrightstarDB on mobile devices. There are a few caveats that you should be aware of (these are called out in the documentation).

* Most critically this has not (yet) been tested on any Android hardware. It works on Android emulators, but real deployment has not been done yet. It is possible that there will be teething problems, especially on devices with restricted memory (see below).
* The library has been compiled with the REST client included but this has not been tested yet
* The persistence layer uses System.IO classes in Mono, not Isolated Storage. This is due to a bug with the implementation of Isolated Storage that prevents us from being able to delete stores. The effect of this is that you need to be careful when specifying the path to the stores directory in the connection string as you will need to provide the (full) path to a storage location where BrightstarDB can read/write and create/delete files.
* The default configuration values for PageCacheSize and QueryCacheMemory are the desktop defaults and certainly too large for memory-constrained devices. This is a critical issue for us to find a way to fix, but the workaround for now is to use set the properties on BrightstarDB.Configuration when your application starts.
* Query is not (yet) optimized for devices with small amounts of memory. Queries that generate many intermediate results (even if these are not in the final results set) can exhaust available memory. This is another area that we plan on working to address in the 1.7 release timescale.

The basic message is that the Android support is there to be experimented with, but should not yet be considered production-ready. However we are releasing it now in its current state in the hope of getting some early feedback to help us get it quickly to the point where it can be used in production.

iOS support is on the roadmap for 1.7.

More Entity Framework Eager Loading
-----------------------------------

When using LINQ to query the BrightstarDB Entity Framework and return a collection of entities as the results, there are two possible paths to go down. The first is to generate a SPARQL query that returns only the IDs of the entities and then to “lazily load” the entities as the client code requests them. This results in N+1 queries to retrieve N entities (one query to find the IDs and then one for each entity to be lazily loaded). The alternative is to generate a SPARQL query that constructs an RDF graph from which the client can then load all of the entities in a single go (so-called “eager loading”). This reduces the query to a single query (albeit one that will generate a much larger results set). Up to now, BrightstarDB has only been able to perform eager loading for queries that return unsorted, unpaged lists of entities. With this update we now support returning sorted, paged lists of entities with eager loading.

This puts much more control into the hands of the client application, as more of the LINQ queries that are written to look like they are eager loading entities are actually now doing that. If you want to stick with the old “lazily loaded” approach, you can write your LINQ query to return the ID of the entities you are interested in, and then use a second LINQ query to retrieve the entity with that ID.

Better Connection Strings for Other Stores
------------------------------------------

On the subject of the Entity Framework we have also revamped the way connection strings are written to connect the BrightstarDB Entity Framework to third-party triple stores. This work enables you to treat a server tha supports multiple stores in much the same way as you would a BrightstarDB server. You can also create a DotNetRDF configuration graph that contains multiple stores. Connections to generic SPARQL endpoints are also much easier to specify now and it is possible to create a read-only connection to a SPARQL query endpoint.

Support for ASP.NET Membership and Role Providers on the Server
---------------------------------------------------------------

The BrightstarDB server can now be configured to authenticate users using an ASP.NET Membership provider and to assign permissions based on the user identity and/or their roles. If you combine this with the [BrightstarDB Membership Provider](/2011/11/adding-a-membershipprovider-to-your-mvc3-application/) we blogged about a while ago you can actually use BrightstarDB to manage the user accounts too.

Support for multiple Export formats
-----------------------------------

The Export Job now supports a parameter to specify the export format. This maybe isn’t such a big step forward right now (we have moved from supporting only NQuads to being able to support NQuads or NTriples…), but we will add other export formats in coming releases (RDF/XML is the first target as well as support for gzip’d output).

NuGet Package Changes
---------------------

We have tidied up the packaging of BrightstarDB on NuGet – it now no longer bundles in dependency assemblies but instead specifies dependencies on other NuGet packages. This makes our package much smaller and makes it play more nicely in large projects where you may have multiple other NuGet package dependencies. We also changed the relationship between the BrightstarDB and the BrightstarDBLibs packages. Previously these two packages both independently included a version of the core BrightstarDB assembly, the only difference between the two being that the BrightstarDB package also included the Text Template file for the Entity Framework and the BrightstarDBLibs package did not. We have now change this so that the BrightstarDB package *only* includes the Text Template file and has a dependency on the BrightstarDBLibs package. In practice this should not make much difference to your projects using BrightstarDB via NuGet, except that you will now see our dependency libraries being included in their own NuGet packages rather than being bundled into ours.

You can download the full installer package from our [Codeplex project page](http://brightstardb.codeplex.com/); update or install via [NuGet](http://www.nuget.org/packages?q=brightstardb) and read the latest docs at [ReadTheDocs.org](http://brightstardb.readthedocs.org/en/latest/).
