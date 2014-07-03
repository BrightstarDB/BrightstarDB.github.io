---
layout: post
title: BrightstarDB Basics: Using SPARQL
tags:
---

SPARQL is a query language for databases that store data in RDF format. Learning to use SPARQL is much the same as learning a query language such as T-SQL, which you would use when querying Microsoft SQL databases. Although at first learning new query languages can be daunting, it is well worth the time invested, as SPARQL is the front runner in querying data that is part of the semantic web.

RDF Format

To understand SPQARL, first it is best to learn more about the format of RDF. RDF stores data in the forms of triples. Each triple consists of a subject, a predicate and an object.

Subject
The subject is the address of the resource being described.
Predicate
The predicate describes the relationship between the subject and the object
Object
The object is the value of the property, this can be either a literal value or the address of another resource

It is sometimes easier to describe these triples as Resource -> Property -> Value

For example, if we were to describe details of a company* in RDF form, it might look similar to the following:
1
2
3
4
<http://business.data.gov.uk/id/company/3477890> <http://research.data.gov.uk/def/project/organisationName> "Meggitt Aerospace Limited" .
<http://business.data.gov.uk/id/company/3477890> <http://research.data.gov.uk/def/project/addressLine1> "Holbrook Lane" .
<http://business.data.gov.uk/id/company/3477890> <http://research.data.gov.uk/def/project/town> "Conventry" .
<http://business.data.gov.uk/id/company/3477890> <http://research.data.gov.uk/def/project/country> "United Kingdom" .
view rawRDF Company Snippet 1 hosted with ❤ by GitHub

*data taken from the business.data.gov.uk dataset on datahub.io, click “view raw” for a cleaner view.
Simple SPARQL Queries

The best way to demonstrate SPARQL queries is to dive right in with some examples. We will be using Polaris tool that installs with BrightstarDB, and the business.data.gov.uk dataset used in the previous post “First steps with Polaris“.

1. Select all businesses ordered by town

select ?business ?town where {?business <http://research.data.gov.uk/def/project/town> ?town } order by ?town
This returns 275 rows of data, consisting of two columns, the first being “business” (the resource ID for the business) and the second being “town”. The results are ordered by the town column, starting with Abbotsley and ending with York.

2. Select all businesses in Manchester

select ?business where {?business <http://research.data.gov.uk/def/project/town> "Manchester" }
SPARQL Query example 2

This query returns 7 rows, consisting of a single column of resource IDs. To return more information about the businesses, we can extend the query to include extra properties and their values.

3. Select all businesses in Manchester and any other information about them.

select * where {?business <http://research.data.gov.uk/def/project/town> "Manchester" . ?business ?property ?value }
SPARQL Query example 3

This returns far more information, in total 150 rows of data are returned that describe the 7 companies returned from the previous query.

This blog post is only designed to give a brief introduction into SPARQL, and so for a more in depth look into the language there are many different resources available on the web.

SPARQL resources online

Full SPARQL 1.1 Documentation at the W3C
Querying semantic data tutorial*
SPARQL By Example*
*written for SPARQL 1.0 but still a thorough introduction to the language