---
layout: post
title: BrightstarDB 1.5 Released
tags: release dotnetrdf jena mono nancy rest xamarin
---

We are pleased to announce that BrightstarDB 1.5 has been released. The new version is now available to download via [NuGet](https://www.nuget.org/packages/brightstardb) or from [our CodePlex site](https://brightstardb.codeplex.com/).

New RESTful HTTP Server
-----------------------

BrightstarDB 1.5 introduces a major change to the BrightstarDB service. In previous releases, the BrightstarDB service was a WCF service. In this new release the WCF service has been replaced by a pure HTTP RESTful service implemented using the [Nancy](http://nancyfx.org/) framework. As with previous releases the REST server can be hosted within IIS or as a Windows Service, but the use of Nancy means that BrightstarDB services can now also be hosted in OWIN-compliant servers such as Katana, and in Apache using mod_mono. The change to using a pure HTTP service also means that BrightstarDB can now better support Mono and can support client-server connections in portable class library applications.

As the service is a simple HTTP service it is also now possible to connect to BrightstarDB from almost any programming or scripting language. We have also put some work into making it possible to work with the service directly from the browser. This is the server home page:

![BrightstarDB REST service homepage]({{site.url}}/assets/bs_rest_homepage.png)

As you can see it shows a list of stores and allows you to create a new store. Clicking on a store takes you to a list of several options for acting on the store. Including a form for running a SPARQL query:

![SPARQL Query Form]({{site.url}}/assets/bs_rest_sparql.png)

And a page for monitoring and creating new jobs:

![BrightstarDB Server Jobs Page]({{site.url}}/assets/bs_rest_jobspage.png)

We have also retained and updated the Polaris desktop application as an alternative way to interact manually with the BrightstarDB service.

Entity Framework support for other RDF stores

The BrightstarDB Entity Framework allows .NET developers to work with their own custom domain models while still providing much of the flexibility of using an RDF triple store. We have now extended that functionality so that Entity Framework code can be executed against a number of other RDF data stores and against any linked data endpoint that supports SPARQL 1.1 Query and Update. In addition to providing data binding functionality, the BrightstarDB entity framework provides comprehensive LINQ-to-SPARQL support, allowing developers to write LINQ queries over their custom domain model that get executed against the RDF triple store as SPARQL queries.

This functionality has been tested against [Jena](http://jena.apache.org/) and using [dotNetRDF](http://dotnetrdf.org/)'s in-memory triple store implementation. In theory the functionality should be able to support any of the RDF stores supported by dotNetRDF as long as they provide (or dotNetRDF can wrap them to provide) a SPARQL 1.1 Query and SPARQL 1.1 Update endpoint. We welcome reports from users on which stores work and which stores do not work with this functionality.

Mono Build

BrightstarDB now builds and runs under [Mono](http://www.mono-project.com/). This has been verified only against version 3.2.4 of Mono (which is the latest stable release at the time of writing). It is know not to work against version 3.2.3 of Mono. We have plans to add support for [Xamarin](http://xamarin.com/) iOS and Android libraries in the near future.

Portable Class Library Client

Previous releases of BrightstarDB were limited to use of embedded BrightstarDB only in the Portable Class Library binaries due to the lack of support for the WCF client library. With the change to using a simple HTTP interface, the Portable Class Library build has been updated so that it is now possible to create applications that connect over HTTP to a remote BrightstarDB server. It is also possible to connect to stores other than BrightstarDB via [dotNetRDF storage connectors](https://bitbucket.org/dotnetrdf/dotnetrdf/wiki/UserGuide/Configuration/Storage%20Providers).

Future Release Plans

It is planned to make a 1.6 release before the end of February 2014. This is scheduled to include support for iOS and Android. You can see the full list of bugs and enhancements at [GitHub](https://github.com/BrightstarDB/BrightstarDB/issues). We are also planning to have at least one intermediate release (which will probably be called 1.5.1) before then. We welcome bug reports and feature requests as well as any other discussion about BrightstarDB on our [CodePlex discussion forum](https://brightstardb.codeplex.com/discussions).