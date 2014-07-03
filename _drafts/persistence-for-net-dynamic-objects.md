---
layout: post
title: Persistence for .NET dynamic objects
tags:
---
The dynamic keyword and related DynamicObject classes were introduced to bridge the gap to COM and into languages such as Python and Ruby. While they offer a lot of power in their own right many people are put off using dynamic objects to build a data driven system because of the lack of a persistence model for dynamic.

BrightstarDB is a NoSQL triple-store database. This means its schema-less and thus an ideal way to store the state of dynamic objects. More specifically BrightstarDB offers a data context dedicated to working with dynamic objects.

To see how the BrightstarDB dynamic support can be used to build a data driven system we have published a complete project on github. In this post we’ll walk through the key aspects.

We start by reviewing the basic support for dynamic persistence in BrightstarDB.

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
47
48
49
50
51
52
53
54
55
56
57
// gets a new BrightstarDB DataObjectContext
var dataObjectContext = BrightstarService.GetDataObjectContext();
 
// create a dynamic context
var dynaContext = new BrightstarDynamicContext(dataObjectContext);
 
// open a new store
var storeId = "DynamicSample" + Guid.NewGuid().ToString();
var dynaStore = dynaContext.CreateStore(storeId);
 
// create some dynamic objects. 
dynamic brightstar = dynaStore.MakeNewObject();
dynamic product = dynaStore.MakeNewObject();
 
// set some properties
brightstar.name = "BrightstarDB";
product.rdfs__label = "Product";
var id = brightstar.Identity;
 
// use namespace mapping (RDF and RDFS are defined by default)
// Assigning a list creates repeated RDF properties.
brightstar.rdfs__label = new[] { "BrightstarDB", "NoSQL Database" };
 
// objects are connected together in the same way
brightstar.rdfs__type = product;
 
dynaStore.SaveChanges();
 
// open store and read some data
dynaStore = dynaContext.OpenStore(storeId);
brightstar = dynaStore.GetDataObject(brightstar.Identity);
 
// property values are ALWAYS collections.
var name = brightstar.name.FirstOrDefault();
Console.WriteLine("Name = " + name);
 
// property can also be accessed by index
var nameByIndex = brightstar.name[0];
Console.WriteLine("Name = " + nameByIndex);
 
// they can be enumerated without a cast
foreach (var l in brightstar.rdfs__label)
{
    Console.WriteLine("Label = " + l);
}
 
// object relationships are navigated in the same way
var p = brightstar.rdfs__type.FirstOrDefault();
Console.WriteLine(p.rdfs__label.FirstOrDefault());
 
// dynamic objects can also be loaded via sparql
dynaStore = dynaContext.OpenStore(storeId);
var objects = dynaStore.BindObjectsWithSparql("select distinct ?dy where { ?dy ?p ?o }");
foreach (var obj in objects)
{
   Console.WriteLine(obj.rdfs__label[0]);
}
view rawdynamic-sample.cs hosted with ❤ by GitHub
The aim of this sample project is to create a dynamic data driven system. While most usages of dynamic are ad-hoc we are building a complete system and want to introduce some structure while only using dynamic objects.

The main thing we want to do is introduce the idea of ‘types’. This may sound a little strange but these are not static types in the .NET sense, they are more like prototypes or type descriptors. Each type will also be a dynamic object and it will contain a known property that will list the names of the keys that apply to instances of this type. We say ‘apply to’ as there is no strict enforcement that instances shall have these properties.

The types are used for grouping and finding instances of the right type and also for dynamically introspecting instances. The other well known property we will use is one that connects instances to their type. To start, each dynamic instance will be connected to just one type, but as this is all dynamic and all ‘just’ data it is possible that an instance could have multiple types. The pattern we are describing here is a classic grounding, or bootstrapping approach to creating layers of structure in a generic model. We use just a couple of well known, or grounding properties, from which we can build structure.

It is important to note that this dynamic system is just one kind. The great thing about dynamic objects is that they let the developer build different kinds of models and meta-models to suit the problem at hand.

Our dynamic system will offer the following interface. The comments on each method provide an outline of what capabilities it provides:

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
47
48
49
50
51
52
53
54
55
56
57
interface IDynamicObjectSystem
{
    /// <summary>
    /// Returns an enumeration of all the 'types' defined in the dynamic system.
    /// </summary>
    IEnumerable<dynamic> Types { get; }
 
    /// <summary>
    /// Either creates a new dynamic object to represent the specified type or updates the instanceProperties
    /// property of the existing type.
    /// </summary>
    /// <param name="name">The unique name of the type</param>
    /// <param name="instanceProperties">A list of strings that defined the allowed properties on an instance of this type</param>
    /// <returns></returns>
    dynamic AssertType(string name, IEnumerable<string> instanceProperties);
 
    /// <summary>
    /// Looks up the type by name and returns it. Returns null if the type does not exist.
    /// </summary>
    /// <param name="name">The name of the type</param>
    /// <returns>A dynamic object for the type or null.</returns>
    dynamic GetType(string name);
 
    /// <summary>
    /// Creates a new persistent and typed dynamic object. 
    /// Throws an exception if the named type does not exist.
    /// </summary>
    /// <returns>A new dynamic object of the specified type.</returns>
    dynamic CreateNewDynamicObject(string typeName);
 
    /// <summary>
    /// Deletes the object with the specified identity. 
    /// </summary>
    /// <param name="id">The identity of the object to delete</param>
    void DeleteObject(string id);
 
    /// <summary>
    /// Commits all changes. 
    /// </summary>
    void SaveChanges();
 
    /// <summary>
    /// Returns an enumeration of all dynamic instances of the specified type.
    /// </summary>
    /// <param name="typeName">The type name</param>
    /// <returns>Enumeration of all instances of the type.</returns>
    IEnumerable<dynamic> GetInstancesOfType(string typeName);
 
    /// <summary>
    /// Uses a SPARQL query to locate a set of objects.
    /// </summary>
    /// <param name="sparqlQuery">The sparql query to execute. Note that the query must have only a single result variable that is 
    /// the id of the object.</param>
    /// <returns>A collection of dynamic objects that meet the query.</returns>
    IEnumerable<dynamic> FindByQuery(string sparqlQuery);
 
}
view rawIDynamicSystem.cs hosted with ❤ by GitHub
The following implementation of the interface shows how we can use the BrightstarDB support for dynamic to implement the methods of our typed dynamic system:

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
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
public class DynamicObjectSystem : IDynamicObjectSystem
{
    private readonly string _connectionString;
    private readonly string _storeId;
    private readonly DynamicStore _dynamicStore;
    public const string UriPrefix = "http://www.brightstardb.com/dynamic/system/";
 
    public DynamicObjectSystem(string connectionString, string storeId)
    {
        _connectionString = connectionString;
        _storeId = storeId;
        var dataObjectContext = BrightstarService.GetDataObjectContext(_connectionString);
        var dynamicContext = new BrightstarDynamicContext(dataObjectContext);
        if (dynamicContext.DoesStoreExist(_storeId))
        {
            _dynamicStore = dynamicContext.OpenStore(_storeId, new Dictionary<string, string>() { { "ds", UriPrefix } });
        }
        else
        {
            _dynamicStore = dynamicContext.CreateStore(_storeId, new Dictionary<string, string>() { { "ds", UriPrefix } });
        }                
    }
 
    private const string TypesQuery = @"select ?id where { ?id <http://www.brightstardb.com/dynamic/system/type> ""type""^^<http://www.w3.org/2001/XMLSchema#string> }";
    private const string GetTypeByNameQuery = @"select ?id where {{ ?id <http://www.brightstardb.com/dynamic/system/name> ""{0}""^^<http://www.w3.org/2001/XMLSchema#string> . ?id <http://www.brightstardb.com/dynamic/system/type> ""type""^^<http://www.w3.org/2001/XMLSchema#string> }}";
    private const string GetInstancesOfTypeQuery = @"select ?id where {{ ?id <http://www.brightstardb.com/dynamic/system/type> <{0}> }}";
 
    public IEnumerable<dynamic> Types
    {
        get { return _dynamicStore.BindObjectsWithSparql(TypesQuery); }
    }
 
    public dynamic AssertType(string name, IEnumerable<string> instanceProperties)
    {
        // check if one exists already
        var type = _dynamicStore.BindObjectsWithSparql(String.Format(GetTypeByNameQuery, name)).FirstOrDefault();
        if (type != null)
        {
            // update instance properties
            type.ds__properties = instanceProperties;
        } else
        {
            // create new type with instance properties
            type = _dynamicStore.MakeNewObject(UriPrefix);
            type.ds__properties = instanceProperties;
            type.ds__name = name;
            type.ds__type = "type";
        }
        _dynamicStore.SaveChanges();
 
        return type;
    }
 
    public dynamic GetType(string name)
    {
        return _dynamicStore.BindObjectsWithSparql(String.Format(GetTypeByNameQuery, name)).FirstOrDefault();           
    }
 
    public void DeleteObject(string id)
    {
        var dataObjectContext = BrightstarService.GetDataObjectContext(_connectionString + ";storename=" + _storeId);
        var store = dataObjectContext.OpenStore(_storeId);
        store.GetDataObject(id).Delete();
        store.SaveChanges();
    }
 
    /// <summary>
    /// Creates a new persistent and typed dynamic object.
    /// </summary>
    /// <returns>A new dynamic object</returns>
    public dynamic CreateNewDynamicObject(string typeName)
    {
        var t = GetType(typeName);
        if (t == null) throw new Exception("Type does not exist");
 
        var d = _dynamicStore.MakeNewObject(UriPrefix);
        d.ds__type = t;
        return d;
    }
 
    public void SaveChanges()
    {
        _dynamicStore.SaveChanges();
    }
 
    public IEnumerable<dynamic> GetInstancesOfType(string typeName)
    {
        var type = GetType(typeName);
        if (type == null) throw new Exception("No type exists for " + typeName);
        return FindByQuery(string.Format(GetInstancesOfTypeQuery, type.Identity));
    }
 
    public IEnumerable<dynamic> FindByQuery(string sparqlQuery)
    {
        return _dynamicStore.BindObjectsWithSparql(sparqlQuery);
    }
}
view rawDynamicSystem.cs hosted with ❤ by GitHub
These tests show how to use the DynamicSystem service object we have created.

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
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
100
101
102
103
104
105
106
107
108
109
110
111
112
113
114
115
116
117
118
119
120
121
122
123
124
125
126
127
128
129
[TestClass]
public class SystemTests
{
    private const string ConnectionString = "type=embedded;storesdirectory=c:\\brightstar";
 
    [TestMethod]
    public void TestCreateObjectSystem()
    {
        var storeId = Guid.NewGuid().ToString();
        var dos = new DynamicObjectSystem(ConnectionString, storeId);
        Assert.IsNotNull(dos);
    }
 
    [TestMethod]
    public void TestNoTypesExist()
    {
        var storeId = Guid.NewGuid().ToString();
        var dos = new DynamicObjectSystem(ConnectionString, storeId);
        var typeCount = dos.Types.Count();
        Assert.AreEqual(0, typeCount);
    }
 
    [TestMethod]
    public void TestAssertType()
    {
        var storeId = Guid.NewGuid().ToString();
        var dos = new DynamicObjectSystem(ConnectionString, storeId);
        var typeCount = dos.Types.Count();
        Assert.AreEqual(0, typeCount);
 
        dos.AssertType("Person", new string[] { "age", "name", "hometown" });
 
        typeCount = dos.Types.Count();
        Assert.AreEqual(1, typeCount);
    }
 
    [TestMethod]
    public void TestGetType()
    {
        var storeId = Guid.NewGuid().ToString();
        var dos = new DynamicObjectSystem(ConnectionString, storeId);
        var typeCount = dos.Types.Count();
        Assert.AreEqual(0, typeCount);
        dos.AssertType("Person", new string[] { "age", "name", "hometown" });
        var type = dos.GetType("Person");
        Assert.IsNotNull(type);
    }
 
    [TestMethod]
    public void TestEnumerateTypeAllowedProperties()
    {
        var storeId = Guid.NewGuid().ToString();
        var dos = new DynamicObjectSystem(ConnectionString, storeId);
        var typeCount = dos.Types.Count();
        Assert.AreEqual(0, typeCount);
        dos.AssertType("Person", new string[] { "age", "name", "hometown" });
        var type = dos.GetType("Person");
 
        var properties = type.ds__properties as IEnumerable<object>;
        Assert.AreEqual(3, properties.ToList().Count());
 
        Assert.IsTrue(properties.Contains("age"));
        Assert.IsTrue(properties.Contains("name"));
        Assert.IsTrue(properties.Contains("hometown"));
    }
 
    [TestMethod]
    public void TestCreateTypedDynamicObject()
    {
        var storeId = Guid.NewGuid().ToString();
        var dos = new DynamicObjectSystem(ConnectionString, storeId);
        var person = dos.AssertType("Person", new string[] { "age", "name", "hometown" });
        var bob = dos.CreateNewDynamicObject("Person");
 
        // check that the id of the type of the instance is the same as the type we passed in.
        Assert.AreEqual(person.Identity, bob.ds__type[0].Identity);
    }
 
    [TestMethod]
    public void TestGetInstancesOfType()
    {
        var storeId = Guid.NewGuid().ToString();
        var dos = new DynamicObjectSystem(ConnectionString, storeId);
        var person = dos.AssertType("Person", new string[] { "age", "name", "hometown" });
        var bob = dos.CreateNewDynamicObject("Person");
        dos.SaveChanges();
 
        var instancesOfPerson = dos.GetInstancesOfType("Person");
        Assert.AreEqual(1, instancesOfPerson.Count());
    }
 
    [TestMethod]
    public void TestDeleteObject()
    {
        var storeId = Guid.NewGuid().ToString();
        var dos = new DynamicObjectSystem(ConnectionString, storeId);
        var person = dos.AssertType("Person", new string[] { "age", "name", "hometown" });
        var bob = dos.CreateNewDynamicObject("Person");
        dos.SaveChanges();
 
        var instancesOfPerson = dos.GetInstancesOfType("Person");
        Assert.AreEqual(1, instancesOfPerson.Count());
 
        // delete object
        dos.DeleteObject(bob.Identity);
        dos.SaveChanges();
 
        // check bob is gone
        instancesOfPerson = dos.GetInstancesOfType("Person");
        Assert.AreEqual(0, instancesOfPerson.Count());
    }
 
    [TestMethod]
    public void TestFindByQuery()
    {
        var storeId = Guid.NewGuid().ToString();
        var dos = new DynamicObjectSystem(ConnectionString, storeId);
        var person = dos.AssertType("Person", new string[] { "age", "name", "hometown" });
        var bob = dos.CreateNewDynamicObject("Person");
        bob.ds__name = "Bob";
        var bill = dos.CreateNewDynamicObject("Person");
        bill.ds__name = "Bill";
        dos.SaveChanges();
 
        var queryResult = dos.FindByQuery("select ?id where { ?id <" + DynamicObjectSystem.UriPrefix + "name> \"Bob\"^^<http://www.w3.org/2001/XMLSchema#string> }" );
        Assert.AreEqual(1, queryResult.Count());
    }
 
}
view rawdynamic-system-tests.cs hosted with ❤ by GitHub
Use cases for a dynamic persistent system

Now that we can easily persist dynamic objects what can we use it for? The generic use case is one where users or administrators or other parts of the system code have a need to create types at runtime. Here are two specific examples:

The first example is a content management system. Administrators are allowed to define content types when the system is running. For each content type they can define the properties that an instance should have. A UI generator can introspect the dynamic types to see which properties to show and instances of each type would of course be dynamic objects.

The second use case is a system component that receives event data from multiple sources. Each source provides consistent data but in a structure that is different from other sources. Some elements of the event data are consistent, such as when it occurred. But all other data is dynamic. The event aggregator implemented using dynamics would be able to create a new dynamic type when it first encouters a new event type and then create dynamic instances to represent the event data.

In this post we have used the basic dynamic persistence capabilities in BrightstarDB to create a complete persistent dynamic system.