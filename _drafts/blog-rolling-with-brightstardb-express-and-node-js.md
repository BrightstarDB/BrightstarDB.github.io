---
layout: post
title: Blog rolling with BrightstarDB, Express and Node.js
tags:
---

Inspired by the use of Node.js and mongoDB in this How To Node article, this tutorial aims to show how BrightstarDB can also be used with Node to create a simple blogging system.


Before starting, ensure you have the following installed;

BrightstarDB
Node.js
ASP.NET MVC
Node needs to be able to access data from BrightstarDB, unfortunately we can’t use WCF under Node, so first of all we will create a simple ASP.NET MVC Rest service which will handle querying and interactions with BrightstarDB.

If you just want to get straight to the code, a download for the source of both projects is available at the end of the post.

Creating MVC Rest interface for BrightstarDB

For the blogging system, our Rest interface will allow the following commands;

Query a store by name using sparql and returning the results as JTriples
Post a new article using N-Triples
To get started, first create a new ASP.NET MVC 3 Project. For the Template, choose Empty.

In the project References add a reference to the NetworkedPlanet.Brightstar.dll.

Add a new Controller to the project and name it StoreController.cs. This will serve as the main interface to the BrightstarDB store. We will be using the BrightstarService class to create an embedded client, so we need to include the following using statement in the top of our controller;

1
using NetworkedPlanet.Brightstar.Client;
view rawStoreController.cs hosted with ❤ by GitHub
Before using the embedded client we need a location for our BrightstarDB data. Open the BrightstarDB management application Polaris and create a new Embedded connection to a location on your computer.

BrightstarDB Embedded Connection
Creating a BrightstarDB Embedded Connection in Polaris
From the Server menu, choose New Store… and call it ‘Blog’.

BrightstarDB Create Store
Create a new store called 'Blog'
This will create a new store in our BrightstarDB data folder named ‘Blog’. Back to our MVC project and StoreController.cs, we can now create an embedded client and point it to the path we used in Polaris to create the ‘Blog’ store.

In StoreController.cs add the following property, be sure to use the location of the BrightstarDB data folder and not the location of the ‘Blog’ store folder;

1
2
3
4
5
6
7
8
9
private IBrightstarService _context;
private IBrightstarService Context
{
    get
    {
        return _context ??
                (_context = BrightstarService.GetEmbeddedClient("C:\\Projects\\node.js\\BrightstarRest\\BSData"));
    }
}
view rawStoreController.cs hosted with ❤ by GitHub
This will create a BrightstarService property called Context which will be using to perform queries and transactions on our ‘Blog’ store.

Next, we first need to be able to query our store, the path to this Rest method will look like this;

/store/{storename}?query=<sparql query>
Add the following method that accepts a storeName and a query variable;

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
22
23
24
25
26
27
28
29
30
// GET: /Store/{storeName}?query=
[HttpGet, ValidateInput(false)]
public ActionResult Store(string storeName, string query)
{
    if (!string.IsNullOrWhiteSpace(query))
    {
        if (Context.DoesStoreExist(storeName))
        {
            Stream queryResult = Context.ExecuteQuery(storeName, query);
            XDocument results = XDocument.Load(queryResult);
            var triples = new List<Dictionary<string, string>>();
            foreach (XElement result in results.SparqlResultRows())
            {
                var columnValues = new Dictionary<string, string>();
                foreach (string name in results.GetVariableNames())
                {
                   // Add the column name and its value to the row dictionary
                   columnValues.Add(name, result.GetColumnValue(name).ToString());
                }
                triples.Add(columnValues);
            }
            return new JsonResult()
                        {
                            Data = triples,
                            JsonRequestBehavior = JsonRequestBehavior.AllowGet
                        };
        }
    }
    return new JsonResult() { Data = Context.DoesStoreExist(storeName), JsonRequestBehavior = JsonRequestBehavior.AllowGet };
}
view rawStoreController.cs hosted with ❤ by GitHub
The above method uses the BrightstarService Context to execute a Sparql query against our named store. The results are iterated by row and the names of the columns and its value added to a Dictionary for each row. A List<Dictionary<string,string>> allows us to return the results using the JsonResult class from ASP.NET in the JTriples format.

Note, the above method uses a Sparql extension method, GetVariableNames, if the method is not available in the release you are using, then add the following code to StoreController.cs and change the above code to use the private mehtod GetVariableNames below.

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
private static readonly XNamespace SparqlResultsNamespace = "http://www.w3.org/2005/sparql-results#";
 
/// <summary>
/// Returns the variable column names in the Sparql result.
/// </summary>
/// <param name="doc">The XDocument containing the result data.</param>
/// <returns>An IEnumerable string of column variable names.</returns>
private static IEnumerable<string> GetVariableNames(XDocument doc)
{
    if (doc.Root != null)
    {
        var head = doc.Root.Element(SparqlResultsNamespace + "head");
        if (head != null)
        {
            return
                head.Elements(SparqlResultsNamespace + "variable").Where(e => (e.Attribute("name") != null)).Select(
                    e => e.Attribute("name").Value);
        }
    }
    return new string[0];
}
view rawStoreController.cs hosted with ❤ by GitHub
To support adding new blog posts, we need to add an Insert method. Our insert method will use the Rest path;

/store/{storename}/insert?triples=<triples>
In StoreController.cs add the following method;

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
// POST: /Store/{storeName}?triples=
[HttpPost, ValidateInput(false)]
public ActionResult Insert(string storeName, string triples)
{
    IJobInfo jobInfo = Context.ExecuteTransaction(storeName, string.Empty, string.Empty, triples, true);
    while (!jobInfo.JobCompletedOk)
    {
        Thread.Sleep(30);
    }
    return new JsonResult() { Data = true };
}
view rawStoreController.cs hosted with ❤ by GitHub
The above method accepts NTriple data and uses the BrightstarService Context to insert the data into the named store. Because we want to return when the job is finished, we wait until the job status is OK before returning. Of course we are assuming that the NTriples are valid and the job will never fail, this may not always be the case so you would need to check for this.

Lastly, to finish the MVC project we need to add additional routes in Global.asax.cs.

Update the RegisterRoutes method with the following;

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
public static void RegisterRoutes(RouteCollection routes)
{
    routes.IgnoreRoute("{resource}.axd/{*pathInfo}");
 
    routes.MapRoute("Store",
                    "store/{storeName}",
                    new { controller = "Store", action = "Store" });
    routes.MapRoute("Insert",
        "store/{storeName}/insert",
        new { controller = "Store", action = "Insert" });
 
    routes.MapRoute(
        "Default", // Route name
        "{controller}/{action}/{id}", // URL with parameters
        new { controller = "Home", action = "Index", id = UrlParameter.Optional } // Parameter defaults
    );
}
view rawGlobal.asax.cs hosted with ❤ by GitHub
This will map our methods in StoreController.cs to the routes specified earlier. Make sure to run the server by pressing F5 and make a note of the port number that the server is using.

Creating the Node blog application

Before starting to write the Node application, first of all we should look at the triples that we will be storing in BrightstarDB for our blog articles.

1
2
3
4
5
6
7
8
@prefix np:<http://www.networkedplanet.com/schema/blog/> .
@prefix dc:<http://purl.org/dc/elements/1.1/> .
@prefix rdf:<http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
 
np:post-id-0 rdf:type np:post .
np:post-id-0 dc:title "post one" .
np:post-id-0 np:body "body one" .
np:post-id-0 dc:date "2012-01-14T10:48:53.280Z" .
view rawgistfile1.txt hosted with ❤ by GitHub
In the above triple we are defining a ‘type’ named ‘post’ that has a ‘title’, ‘body’ and ‘date’. For the sake of simplicity we will exclude storing comments in this post. To get started with the Node application, as well as Node installed, we need to install Express. To do this, execute the following in a command prompt;

npm install express -g
Note: Installing Nodejs under Windows should add node and npm to the system path, if it doesn’t add the following to your Path system environment variables;

C:\Users\*username*\AppData\Roaming\npm;C:\Program Files (x86)\nodejs\;
With Express installed, we can now create an Express starter project. In a command prompt run the following;

mkdir blog cd blog express -c stylus npm install -d
This will generate an Express application using the stylus css engine, and download any dependancies using npm. To test that the application generated successfully, run node against the main app.js;

node app.js
You should now be able to access localhost:3000 and see the default express content. We will be creating the following operations in our blog application, these are:

Displaying all blog articles
Creating a blog article
Displaying an individual blog article
To create the interface between our application and the BrightstarDB ASP.NET MVC  service created earlier, we will add an communication layer named brightstar-server.js to our app.

Create a new file under the /blog folder named ‘brightstar-server.js’ and add the following;

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
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
var util = require('util'),
    http = require('http');
 
BrightstarServer = function (host, port, store) {
    this.host = host,
    this.port = port,
    this.path = "/store/" + store;
};
 
BrightstarServer.prototype = {
    /* Defaults */
    host: "localhost",
    port: 88,
    path: "/store/",
    collection: function (item, callback) {
        console.log("Fetching all articles...");
        var articlesQuery = "select ?_id ?title ?body ?created_at where { ?_id <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.networkedplanet.com/schema/blog/post> . ?_id <http://purl.org/dc/elements/1.1/title> ?title . ?_id <http://www.networkedplanet.com/schema/blog/body> ?body . ?_id <http://purl.org/dc/elements/1.1/date> ?created_at }",
            options = {
                host: this.host,
                port: this.port,
                path: this.path
            };
        options.path = options.path + "/?query=" + encodeURIComponent(articlesQuery);
        var req = http.request(options, function (res) {
            res.setEncoding('utf8');
            res.on('data', function (chunk) {
                var jsonData = JSON.parse(chunk),
                    i;
                for (i = 0; i < jsonData.length; i++) {
                    jsonData[i]._id = jsonData[i]._id.replace(/http:\/\/www.networkedplanet.com\/schema\/blog\//i, '');
                    jsonData[i].created_at = new Date(jsonData[i].created_at);
                }
                console.log("Returning articles");
                callback(null, jsonData);
            });
        });
        req.end();
    }
};
 
exports.BrightstarServer = BrightstarServer;
view rawbrightstar-server.js hosted with ❤ by GitHub
The BrightstarServer class provides a single method collection, that will contact the BrightstarDB ASP.NET MVC server using an HTTP request, execute a Sparql query to return all blog posts and process the resulting JSON. The Sparql query is requesting the ‘_id’, ‘title’, ‘body’, and ‘created_at’ properties for each blog post. The JSON returned will look similar to the data below;

1
2
3
4
5
6
[{
  _id: 'post-id-0',
  title: '',
  body: '',
  created_at: new Date()
}]
view rawgistfile1.json hosted with ❤ by GitHub
Next we will create an articleprovider that uses the brightstarserver class. Add a new file named ‘articleprovider-brightstar.js’ and include the following;

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
var BsS = require('./brightstar-server').BrightstarServer;
ArticleProvider = function (host, port, store) {
    this.db = new BsS(host, port, store);
};
 
ArticleProvider.prototype.findAll = function (callback) {
  this.db.collection(function (error, articleCollection) {
    if (error) callback(error);
    else {
      callback(null, articleCollection);
    }
  });
};
 
exports.ArticleProvider = ArticleProvider;
view rawarticleprovider-brightstar.js hosted with ❤ by GitHub
The ArticleProvider provides a single method findAll that uses the BrightstarServer class to fetch all of the blog posts. To make our blog app uses our ArticleProvider, we need to update app.js to render a view when the ‘/’ route is called and display all of the blog posts.

Update app.js with the following ensuring that the port number for the ASP.NET server earlier is used when initialising ArticleProvider;

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
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
/**
* Module dependencies.
*/
 
var express = require('express'), routes = require('./routes');
var ArticleProvider = require('./articleprovider-brightstar').ArticleProvider;
 
var app = module.exports = express.createServer();
 
// Configuration
 
app.configure(function () {
    app.set('views', __dirname + '/views');
    app.set('view engine', 'jade');
    app.use(express.bodyParser());
    app.use(express.methodOverride());
    app.use(require('stylus').middleware({ src: __dirname + '/public' }));
    app.use(app.router);
    app.use(express.static(__dirname + '/public'));
});
 
app.configure('development', function () {
    app.use(express.errorHandler({ dumpExceptions: true, showStack: true }));
});
 
app.configure('production', function () {
    app.use(express.errorHandler());
});
 
// Routes
 
var articleProvider = new ArticleProvider('localhost', 54543, 'blog');
 
app.get('/', function (req, res) {
    articleProvider.findAll(function (error, docs) {
        res.render('index.jade', {
            locals: {
                title: 'Blog',
                articles: docs
            }
        });
    });
});
 
app.listen(3000);
console.log("Express server listening on port %d in %s mode", app.address().port, app.settings.env);
view rawapp.js hosted with ❤ by GitHub
Our application is now using the BrightstarDB ArticleProvider, calling its findAll method to fetch all of the blog posts and passing them to a view ‘index.jade’, which we will add now. Create a new file named index.jade and add the following code;

1
2
3
4
5
6
7
8
h1= title
#articles
    - each article in articles
      div.article
        div.created_at= article.created_at
        div.title
            a(href="/blog/"+article._id)!= article.title
        div.body= article.body
view rawindex.jade hosted with ❤ by GitHub
Update the file style.styl the following css rules;

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
22
23
24
25
26
27
28
body
  font-family "Helvetica Neue", "Lucida Grande", "Arial"
  font-size 13px
  text-align center
  text-stroke 1px rgba(255, 255, 255, 0.1)
  color #555
h1, h2
  margin 0
  font-size 22px
  color #343434
h1
  text-shadow 1px 2px 2px #ddd
  font-size 60px
#articles
  text-align left
  margin-left auto
  margin-right auto
  width 320px
  .article
    margin 20px
    .created_at
        display none
    .title
        font-weight bold
        text-decoration underline
        background-color #eee
    .body
        background-color #ffa
view rawstyle.styl hosted with ❤ by GitHub
Before we can test that the app is working correctly and fetching blog posts, we need to populate our store with some test data. Using Polaris, run a new transaction on the ‘Blog’ store using the following test data;

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
<http://www.networkedplanet.com/schema/blog/post-id-0> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.networkedplanet.com/schema/blog/post> .
<http://www.networkedplanet.com/schema/blog/post-id-0> <http://purl.org/dc/elements/1.1/title> "post one" .
<http://www.networkedplanet.com/schema/blog/post-id-0> <http://www.networkedplanet.com/schema/blog/body> "body one" .
<http://www.networkedplanet.com/schema/blog/post-id-0> <http://purl.org/dc/elements/1.1/date> "2012-01-14T10:48:53.280Z" .
<http://www.networkedplanet.com/schema/blog/post-id-1> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.networkedplanet.com/schema/blog/post> .
<http://www.networkedplanet.com/schema/blog/post-id-1> <http://purl.org/dc/elements/1.1/title> "post two" .
<http://www.networkedplanet.com/schema/blog/post-id-1> <http://www.networkedplanet.com/schema/blog/body> "body two" .
<http://www.networkedplanet.com/schema/blog/post-id-1> <http://purl.org/dc/elements/1.1/date> "2012-01-16T10:49:53.280Z" .
<http://www.networkedplanet.com/schema/blog/post-id-2> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.networkedplanet.com/schema/blog/post> .
<http://www.networkedplanet.com/schema/blog/post-id-2> <http://purl.org/dc/elements/1.1/title> "post three" .
<http://www.networkedplanet.com/schema/blog/post-id-2> <http://www.networkedplanet.com/schema/blog/body> "body three" .
<http://www.networkedplanet.com/schema/blog/post-id-2> <http://purl.org/dc/elements/1.1/date> "2012-01-18T10:50:53.280Z" .
view rawgistfile1.txt hosted with ❤ by GitHub
BrightstarDB Add Triples using Polaris
Add Triples using Polaris
With the test data added, restart the Node server and run node app.jsagain. Refresh the site and there should now be three blog posts displayed on the page.

Blog test data
Blog first page showing our test data
Now that we can display our posts we need to add functionality to create new articles. Create a new view named blog_new.jade and the following code;

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
h1= title
form( method="post")
    div
        div
            span Title :
            input(type="text", name="title", id="editArticleTitle")
        div
            span Body :
            textarea( name="body", rows=20, id="editArticleBody)
        div#editArticleSubmit
            input(type="submit", value="Send")
view rawblog_new.jade hosted with ❤ by GitHub
In app.js add another two routes to handle creating new blog posts;

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
app.get('/blog/new', function (req, res) {
    res.render('blog_new.jade', {
        locals: {
            title: 'New Post'
        }
    });
});
 
app.post('/blog/new', function (req, res) {
    articleProvider.save({
        title: req.param('title'),
        body: req.param('body')
    }, function (error, docs) {
        res.redirect('/');
    });
});
view rawapp.js hosted with ❤ by GitHub
In brightstar-server.js and articleprovider-brightstar.js we need to add the the save methods to handle saving blog posts. Open brightstar-server.js and add the following method after the collection method;

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
22
23
24
    save: function (item, callback) {
        console.log("Saving article...");
        var options = {
                host: this.host,
                port: this.port,
                path: this.path,
                method: 'POST'
            },
            postId = "&lt;http://www.networkedplanet.com/schema/blog/post-id-" + item._id + "&gt;",
            triple = postId + " &lt;http://www.w3.org/1999/02/22-rdf-syntax-ns#type&gt; &lt;http://www.networkedplanet.com/schema/blog/post&gt; .\n";
        triple = triple + postId + " &lt;http://purl.org/dc/elements/1.1/title&gt; \"" + item.title + "\" .\n";
        triple = triple + postId + " &lt;http://www.networkedplanet.com/schema/blog/body&gt; \"" + item.body + "\" .\n";
        triple = triple + postId + " &lt;http://purl.org/dc/elements/1.1/date&gt; \"" + item.created_at.toISOString() + "\" .";
        options.path = options.path + "/insert/?triples=" + encodeURIComponent(triple);
        var req = http.request(options, function (res) {
            res.setEncoding('utf8');
            res.on('data', function (chunk) {
                var jsonData = JSON.parse(chunk);
                console.log("Article saved");
                callback(null, jsonData);
            });
        });
        req.end();
    }
view rawbrightstar-server.js hosted with ❤ by GitHub
The save method creates a set of NTriples from our post object and sends them to our ASP.NET MVC Rest server. Next we need to add the save method to our ArticleProvider.

In articleprovider-brightstar.js add another method for saving posts;

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
ArticleProvider.prototype.save = function (article, callback) {
    var that = this;
    this.findAll(function (error, articleCollection) {
        if (error) callback(error);
        article.created_at = new Date();
        if (article.comments === undefined) article.comments = [];
        for (var j = 0; j &lt; article.comments.length; j++) {
            article.comments[j].created_at = new Date();
        }
        article._id = articleCollection.length;
        that.db.save(article, function () {
            callback(null, article);
        });
    });
};
view rawarticleprovider-brightstar.js hosted with ❤ by GitHub
In the above method we first get all of the posts using the findAll method and use the number of existing posts to create an Id, then pass the article to BrightstarServer to save the post to the server. Reload the Node server and navigate to localhost:3000/blog/new and check that posts can be saved. Note, since characters are not escaped when saving the posts as triples, using carriage returns will cause an error!

Finally, we will add the ability to view a single post by Id. Open brightstar-server.js and add another method beneath the save method added previously;

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
22
23
24
    findById: function (id, callback) {
        var articleQuery = "select ?title ?body ?created_at where { <http://www.networkedplanet.com/schema/blog/{{id}}> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.networkedplanet.com/schema/blog/post> . <http://www.networkedplanet.com/schema/blog/{{id}}> <http://purl.org/dc/elements/1.1/title> ?title . <http://www.networkedplanet.com/schema/blog/{{id}}> <http://www.networkedplanet.com/schema/blog/body> ?body . <http://www.networkedplanet.com/schema/blog/{{id}}> <http://purl.org/dc/elements/1.1/date> ?created_at }",
            options = {
                host: this.host,
                port: this.port,
                path: this.path
            };
        options.path = options.path + "/?query=" + encodeURIComponent(articleQuery.replace(/\{\{id\}\}/g, id));
        console.log("Finding post by Id:" + id);
        var req = http.request(options, function (res) {
            res.setEncoding('utf8');
            res.on('data', function (chunk) {
                var jsonData = JSON.parse(chunk),
                    i;
                for (i = 0; i < jsonData.length; i++) {
                    jsonData[i].created_at = new Date(jsonData[i].created_at);
                    jsonData[i].comments = [];
                }
                console.log("Returning post");
                callback(null, jsonData[0]);
            });
        });
        req.end();
    },
view rawbrightstar-server.js hosted with ❤ by GitHub
In the method above we are using another Sparql query to return the ‘title’, ‘body, and ‘created_at’ properties from the given Id. Since we are storing the ISO date, we need to create a Date() object using the date string for each row.

Next update articleprovider-brightstar.js with the following code to fetch a single post;

1
2
3
4
5
6
7
8
ArticleProvider.prototype.findById = function (id, callback) {
    this.db.findById(id, function (error, article) {
        if (error) callback(error);
        else {
            callback(null, article);
        }
    });
};
view rawarticleprovider-brightstar.js hosted with ❤ by GitHub
In the  code above we are simply calling the BrightstarServer method findById. Lastly, we need to add a new view to display a single article and add a route in app.js.

Create a new view and name it blog_show.jade, and add the following code;

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
h1= title
div.article
    div.created_at= article.created_at
    div.title= article.title
    div.body= article.body
    - each comment in article.comments
      div.comment
        div.person= comment.person
        div.comment= comment.comment
    div
      form( method="post", action="/blog/addComment")
        input( type="hidden", name="_id", value=article._id.toHexString())
        div
          span Author :
          input( type="text", name="person", id="addCommentPerson")
        div
          span Comment :
          textarea( name="comment", rows=5, id="addCommentComment")
        div#editArticleSubmit
          input(type="submit", value="Send")
view rawblog_show.jade hosted with ❤ by GitHub
In app.js add another route for viewing individual posts;

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
app.get('/blog/:id', function(req, res) {
    articleProvider.findById(req.params.id, function(error, article) {
        res.render('blog_show.jade',
        { locals: {
            title: article.title,
            article:article
        }
        });
    });
});
view rawapp.js hosted with ❤ by GitHub
Restart the Node server and we should now be able to view all posts, open individual posts, and create new posts. It should also be possible to add comments to the blogging system so adding comments has been left as an exercise for the reader.

Hopefully this shows the possibilities that are available in using Node.js and BrightstarDB together.

Download the source for both projects.
BrightstarNodeJS-source