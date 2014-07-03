---
layout: post
title: BrightstarDB Basics: A Simple code-first .NET Entity Framework project
tags:
---

Let’s build a quick and simple .NET project to show how to create and update data using the BrightstarDB code-first entity framework.

First, open Visual Studio and create a new console application; then add a reference to the BrightstarDB DLL that is found in the installation directory (under BrightstarDB/SDK/NET40). Make sure that the Target framework is not set to a client profile. Register your license as outlined in the documentation “Licensing For Embedded Applications”

Entities are Interfaces

Each entity in our system is created using an Interface. Right click the project and select Add > New Item and select the Interface filetype; give it the name IPerson. Repeat this step and create another interface named ICompany.

To turn these interfaces into BrightstarDB entities, we add the Entity attribute to the interface. Every BrightstarDB entity must have a read-only Id property that is of type String. After both interfaces have the Entity attribute and the Id property, any number of properties can then be added to the entities, as shown below:
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
using BrightstarDB.EntityFramework;
 
namespace BrightstarApp
{
    [Entity]
    public interface ICompany
    {
        string Id { get; }
        string Name { get; set; }
        string Email { get; set; }
        string Telephone { get; set; }
        string Street { get; set; }
        string Locality { get; set; }
        string Region { get; set; }
        string Country { get; set; }
        string PostalCode { get; set; }
    }
}
view rawICompany hosted with ❤ by GitHub
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
using System;
using BrightstarDB.EntityFramework;
 
namespace BrightstarApp
{
    [Entity]
    public interface IPerson
    {
        string Id { get; }
        string FirstName { get; set; }
        string LastName { get; set; }
        DateTime DateOfBirth { get; set; }
        string Email { get; set; }
        string Website { get; set; }
    }
}
view rawIPerson hosted with ❤ by GitHub
 
Relationships between Entities

To link the Person and Company entities, simply add a property to each of the entities showing the relationship between the two.
1
2
3
4
5
6
7
    [Entity]
    public interface ICompany
    {
        string Id { get; }
        ...
        ICollection<IPerson> Employees { get; set; }  
    }
view rawICompany hosted with ❤ by GitHub
1
2
3
4
5
6
7
8
    [Entity]
    public interface IPerson
    {
        string Id { get; }
        ...
        [InverseProperty("Employees")]
        ICompany Company { get; set; }
    }
view rawIPerson hosted with ❤ by GitHub

The second property is given the InverseProperty attribute, so that it is marked as describing the other end of the same relationship.
BrightstarDB Context file

Right click on the project and select Add > New Item > Brightstar Entity Context (in Data category). This adds a Text Transform file which picks up on the interfaces with the Entity attribute and builds a BrightstarEntityContext from the code. If you make any changes to the code of the interfaces, you can rebuild the BrightstarDB Context at any time by right clicking on the context file and selecting “Run Custom Tool”.

Creating and updating entities

Open the Program.cs file and add the following code:
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
using System;
 
namespace BrightstarApp
{
    class Program
    {
        static void Main(string[] args)
        {
            var storeName = "SimpleStore_" + DateTime.Now.ToString("yyyyMMddHHmmss");
            //connection string to the BrightstarDB service
            string connectionString =
                string.Format(@"Type=http;endpoint=http://localhost:8090/brightstar;StoreName={0};", storeName);
 
            var ctx = new MyEntityContext(connectionString);
 
            var t = ctx.Persons.Create();
            t.FirstName = "Tim";
            t.LastName = "Bisley";
            t.DateOfBirth = new DateTime(1979, 3, 12);
            t.Email = "tim@fantasybazaar.co.uk";
            t.Website = "http://doktormandrake.tumblr.com";
 
            var d = ctx.Persons.Create();
            d.FirstName = "Daisy";
            d.LastName = "Steiner";
            d.DateOfBirth = new DateTime(1980, 6, 24);
            d.Email = "happydaze@citytemps.co.uk";
            d.Website = "http://boglingtoaswad.blogspot.com";
 
            var bb = ctx.Persons.Create();
            bb.FirstName = "Bilbo";
            bb.LastName = "Bagshot";
            bb.Email = "theboss@fantasybazaar.co.uk";
 
            ctx.SaveChanges();
 
        }
    }
}
view rawcreate entities hosted with ❤ by GitHub
1
2
3
4
5
6
7
8
9
var fb = ctx.Companies.Create();
fb.Name = "Fantasy Bazaar";
fb.Email = "info@fantasybazaar.co.uk";
fb.Locality = "Tufnell Park";
fb.Region = "London";
fb.Employees.Add(bb);
fb.Employees.Add(t);
 
ctx.SaveChanges();
view rawgistfile1.txt hosted with ❤ by GitHub
This creates 3 Person entities and adds properties to them. It also shows the creation of a Company entity and links that entity to two of the Person entities. Run the program (F5) to create the
Reading entities

To quickly view the data produced by this code, you can open Polaris and do a quick query. To access entities problematically you can use LINQ similar to the example below:
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
ctx = new MyEntityContext(connectionString);
var shop = ctx.Companies.FirstOrDefault(c => c.Name.Equals("Fantasy Bazaar"));
if (shop != null)
{
	Console.WriteLine("Employees at '{0}'", shop.Name);
	foreach (var e in shop.Employees)
	{
	    Console.WriteLine("> " + e.FirstName + " " + e.LastName);
	}
}
Console.ReadLine();
view rawwrite out employees hosted with ❤ by GitHub
More information

For more detailed explanations of how to develop with BrightstarDB, read Developing with BrightstarDB. There are also samples in the BrightstarDB installer, and explanations for each sample within the BrightstarDB documentation. As always you can use the BrightstarDB Community Pages for any questions or feedback.