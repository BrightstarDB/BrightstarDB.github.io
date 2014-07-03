---
layout: post
title: BrightstarDB Basics: First steps with Polaris
tags:
---
Creating a connection

When opening Polaris for the first time, you will be given a prompt to create a connection:

Polaris promptWithout a connection set up you will not be able to create any stores, so click “Yes” to open a new Connection Properties dialog box:

New connection in Polaris

The recommended setting is to connect to a BrightstarDB server; selecting “HTTP” from the Connection Type field will auto-fill the other fields with the default settings for a local BrightstarDB service. Add a name for the connection and click “OK”.

Note: the BrightstarDB installation sets up the BrightstarDB service at the localhost:8090 HTTP endpoint. You must have the IIS feature enabled on your machine for this endpoint to have been created successfully. If after creating a connection, Polaris reports it is not able to connect, check the installation pre-requisites and that the service is has started successfully.

Creating a store

Creating a new store within Polaris simply consists of right-clicking on the connection name, and selecting “New Store”

New Store

Adding data

There are a number of different ways to add data to a store, which you choose will depend entirely on your project. For the purpose of this blog post we will find an existing dataset on the web, and import it directly into our new BrightstarDB store.

I have chosen a widely available dataset of business-related data from data.gov.uk, which can be dowloaded from the datahub.io. Right click on the store, and select New > Import Job Import the dataset into your newly created store (for more information, see the “Importing Data” section of Using Polaris)

Import Data into BrightstarDB

Import Success

Query the data

Querying in BrightstarDB is done with SPARQL. Detailing SPARQL queries is outside the scope of this blog post, but the most basic one would be a “select all”:

SELECT * WHERE {?subject ?predicate ?object}
Querying data with SPARQL

The next post will talk more about SPARQL querying of data, or use the BrightstarDB Community Pages for any questions or feedback.