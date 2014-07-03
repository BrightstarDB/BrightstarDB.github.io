---
layout: post
title: Enable SPARQL Query in your MVC3 Application
tags: sparql mvc
---

NOTE: Update this for more uptodate MVC!

BrightstarDB uses SPARQL as its primary query language. Because of this and because all the entities you create with the BrightstarDB entity framework are RDF resources, it is possible to turn your application into a part of the Linked Data web with just a few lines of code. The easiest way to achieve this is to add a controller for running SPARQL queries.

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
using System;
using System.Web.Mvc;
using NetworkedPlanet.Brightstar.Client;
 
namespace NetworkedPlanet.BrightStar.Samples.NerdDinner.Controllers
{
    public class SparqlController : Controller
    {
        [ValidateInput(false)]
        public ActionResult Index(string query)
        {
            if (String.IsNullOrEmpty(query))
            {
                return View("Error");
            }
            var client = BrightstarService.GetClient();
            var results = client.ExecuteQuery("NerdDinner", query);
            return new FileStreamResult(results, "application/xml; charset=utf-16");
        }
    }
}
view rawSparqlController.cs hosted with ❤ by GitHub
As shown in the code above, all you need to do in the code is create a client instance (when using no parameters like this, the connection string is read from your web.config file). The client you get back is actually a BrightstarDB service client and is not tied to the store specified in your connection string, so you need to provide the store name as a parameter to ExecuteQuery (in this case “NerdDinner”). The result of the ExecuteQuery method is a stream containing the SPARQL results in XML format, so all you need to do is wrap this stream in a FileStreamResult and set an appropriate media type. The [ValidateInput(false)] decoration is required because SPARQL queries contain characters such as < and > which ASP.NET considers harmful, if you omit this attribute you will get an exception thrown from ASP.NET.

Now you can query the SPARQL endpoint like this:

http://localhost:49608/sparql?query=SELECT ?d ?o WHERE {?d a <http://brightstardb.com/namespaces/default/Dinner>}