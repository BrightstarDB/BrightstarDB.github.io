---
layout: post
title: The Joy of Testing with an Embeddable Database
tags:
---
One of the really powerful things about having a database that can create new instances without any heavy installation means that it’s possible to create a great suite of tests for an application’s data layer. In essence every test that runs can create its own database instance, populate it with just the right data for a test and then execute.

In BrightstarDB a connection string has the magic property of creating a database instance if the named store does not exist. The following example shows a connection string to an embeddeded instance called ‘Test1′. (The same magic also works if connecting to a remote BrightstarDB instance).

1
"type=embedded;storesdirectory=c:\brightstar;storename=Test1"
view rawConnectionString hosted with ❤ by GitHub
This can be used when creating tests so that each test has its own store to work with. The following example shows a template test class that provides a new connection string and store for each test. It uses a GUID for the store name so that each test receives a unique store.

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
    [TestClass]
    public class EntityContextTests
    {
        private const string ConnectionString = "type=embedded;storesdirectory=c:\\brightstar";
 
        private static MyEntityContext GetContext(string storeId)
        {
            return new MyEntityContext(ConnectionString + ";storename=" + storeId); 
        }
 
        [TestMethod]
        public void TestNumber1()
        {
            // The context is a new context to a brand new store
            var ctx = GetContext(Guid.NewGuid().ToString());
 
            // context operations for the test 
        }
    }
view rawSampleTest.cs hosted with ❤ by GitHub
The sample above is working with a BrightstarDB Entity Framework context, but any additional layer that a developer has built on top of the EF context can be tested in the same way. A good example of this can be seen in the EventFeedModeland related tests that we published.

It’s extremely liberating to consider each database instance as disposable. In the past database instances have felt a little too persistent. Setting up scripts to clean out a database is possible but required a bunch of work to make happen. Quite simply this was enough of a barrier to prevent real unit testing from taking place. Removing the pain and inconvenience around running tests against a peristent store means writing tests for the data layer is just as straight forward as for any normal set of classes. This enables developers to get greater test coverage of, and confidence in, a key part of their application.