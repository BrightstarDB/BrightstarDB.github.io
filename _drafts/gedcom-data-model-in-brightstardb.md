---
layout: post
title: GEDCOM Data Model in BrightstarDB
tags:
---

One request we had for a BrightstarDB data model project was to provide a simple model to represent genealogy information. As well as the core data model the project should be able to consume data in the GEDCOM format. GEDCOM (GEnealogical Data COMmunication) is a specification for exchanging genealogical data between different genealogy software [ref]. As always the complete project source code is freely available from a BrightstarDB repository.

The model I created would only support a subset of GEDCOM, but it would be enough to describe individuals, marriages and children. This would be enough to represent most family trees. The following diagram shows the core types and relationships.



This model was then implemented using the following C# interface definitions and BrightstarDB Entity Framework annotations.

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
namespace BrightstarDB.Gedcom
{
    [Entity]
    public interface IFamily
    {
        IMarriageEvent MarriageEvent { get; set; }
        IIndividual Husband { get; set; }
        IIndividual Wife { get; set; }
        ICollection<IIndividual> Children { get; set; }
    }
 
    [Entity]
    public interface IMarriageEvent
    {
        string Place { get; set; }
        string Date { get; set; }
    }
 
    [Entity]
    public interface IIndividual
    {
        string Id { get; }
        string Name { get; set; }
        string Sex { get; set; }
        IBirthEvent BirthEvent { get; set; }
        IDeathEvent DeathEvent { get; set; }
 
        IEnumerable<IFamily> SpouseFamilies();
    }
 
    [Entity]
    public interface IBirthEvent
    {
        string Place { get; set; }
        string Date { get; set; }
    }
 
    [Entity]
    public interface IDeathEvent
    {
        string Place { get; set; }
        string Date { get; set; }
    }
    
}
view rawGEDCOMInterfaces.cs hosted with ❤ by GitHub
To process the GEDCOM format into this model was a two step process. First of all I used the code by Aaron Skonnard in his MSDN article XML Data Migration Case Study: GEDCOM to turn the raw GEDCOM data into XML. I then wrote a processor that would take the XML and create data model objects.

The processor has the following implementation.

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
130
131
132
133
134
135
136
137
138
139
140
141
142
143
144
145
146
147
148
149
150
151
152
153
154
155
156
157
158
159
160
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Xml.Linq;
 
namespace BrightstarDB.Gedcom
{
    public class GedcomImporter
    {
        public static void Import(string fileName, string connectionString)
        {
            // parse the gedcom
            var gedcomParser = new GedcomReader(fileName);
            var d = XDocument.Load(gedcomParser);
 
            // create a context and create the entities.
            var ctx = new GedComContext(connectionString);
 
            var lookups = new Dictionary<string, object>();
 
            foreach (var elem in d.Root.Elements())
            {
                if (elem.Name.LocalName.Equals("INDI"))
                {
                    var id = TryGetValue(elem, "id");
                    var individual = ctx.Individuals.Create();
                    if (id != null) lookups[id] = individual;
 
                    foreach (var elem1 in elem.Elements())
                    {
                        if (elem1.Name.LocalName.Equals("NAME"))
                        {
                            var name = TryGetValue(elem1, "value");
                            if (name != null) individual.Name = name;
                        }
 
                        if (elem1.Name.LocalName.Equals("SEX"))
                        {
                            var sex = TryGetValue(elem1, "value");
                            if (sex != null) individual.Sex = sex;
                        }
 
                        if (elem1.Name.LocalName.Equals("BIRT"))
                        {
                            var birth = ctx.BirthEvents.Create();
                            individual.BirthEvent = birth;
                            foreach (var elem2 in elem1.Elements())
                            {
                                if (elem2.Name.LocalName.Equals("PLAC"))
                                {
                                    var place = TryGetValue(elem2, "value");
                                    if (place != null) birth.Place = place;
                                }
 
                                if (elem2.Name.LocalName.Equals("DATE"))
                                {
                                    var date = TryGetValue(elem2, "value");
                                    if (date != null) birth.Date = date;
                                }
                            }
                        }
 
                        if (elem1.Name.LocalName.Equals("DEAT"))
                        {
                            var death = ctx.DeathEvents.Create();
                            individual.DeathEvent = death;
                            foreach (var elem2 in elem1.Elements())
                            {
                                if (elem2.Name.LocalName.Equals("PLAC"))
                                {
                                    var place = TryGetValue(elem2, "value");
                                    if (place != null) death.Place = place;
                                }
 
                                if (elem2.Name.LocalName.Equals("DATE"))
                                {
                                    var date = TryGetValue(elem2, "value");
                                    if (date != null) death.Date = date;
                                }
                            }
                        }
                    }
 
                }
 
                if (elem.Name.LocalName.Equals("FAM"))
                {
                    var id = TryGetValue(elem, "id");
                    var family = ctx.Families.Create();
                    if (id != null) lookups[id] = family;
 
                    foreach (var elem1 in elem.Elements())
                    {
                        if (elem1.Name.LocalName.Equals("MARR"))
                        {
                            var marriage = ctx.MarriageEvents.Create();
                            family.MarriageEvent = marriage;
                            foreach (var elem2 in elem1.Elements())
                            {
                                if (elem2.Name.LocalName.Equals("PLAC"))
                                {
                                    var place = TryGetValue(elem2, "value");
                                    if (place != null) marriage.Place = place;
                                }
 
                                if (elem2.Name.LocalName.Equals("DATE"))
                                {
                                    var date = TryGetValue(elem2, "value");
                                    if (date != null) marriage.Date = date;
                                }
                            }
                        }
 
                        if (elem1.Name.LocalName.Equals("HUSB"))
                        {
                            var idref = TryGetValue(elem1, "idref");
                            if (idref != null)
                            {
                                var husb = lookups[idref] as IIndividual;
                                if (husb != null) family.Husband = husb;
                            }
                        }
 
                        if (elem1.Name.LocalName.Equals("WIFE"))
                        {
                            var idref = TryGetValue(elem1, "idref");
                            if (idref != null)
                            {
                                var wife = lookups[idref] as IIndividual;
                                if (wife != null) family.Wife = wife;
                            }
                        }
 
                        if (elem1.Name.LocalName.Equals("CHIL"))
                        {
                            var idref = TryGetValue(elem1, "idref");
                            if (idref != null)
                            {
                                var child = lookups[idref] as IIndividual;
                                if (child != null) family.Children.Add(child);
                            }
                        }
                    }
 
                }
            }
            ctx.SaveChanges();
        }
 
        private static string TryGetValue(XElement elem, string attributeName)
        {
            if (elem.Attribute(attributeName) != null)
            {
                return elem.Attribute(attributeName).Value;
            }
            return null;
        }
    }
}
view rawGEDCOMImporter.cs hosted with ❤ by GitHub
As well as the core data model I added an extra method to Individual.cs called SpouseFamilies(). This method would find any families in which the indidual was either the Husband or the Wife. This would enable users of the the model to display a complete tree. Given an individual it would be possible to find the families where they were one of the spouses. The family would then link to their children. The method could then be used on each child individual to see in which families they were the spouse, and so on.

With the processor and model complete I tested it against some simple GEDCOM data. The following GEDCOM data was processed:

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
0 HEAD
1 CHAR ASCII
1 SOUR ID_OF_CREATING_FILE
1 GEDC
2 VERS 5.5
2 FORM Lineage-Linked
1 SUBM @SUBMITTER@
0 @SUBMITTER@ SUBM
1 NAME /Submitter/
1 ADDR Submitters address
2 CONT address continued here
0 @FATHER@ INDI
1 NAME /Father/
1 SEX M
1 BIRT
2 PLAC birth place
2 DATE 1 JAN 1899
1 DEAT
2 PLAC death place
2 DATE 31 DEC 1990
1 FAMS @FAMILY@
0 @MOTHER@ INDI
1 NAME /Mother/
1 SEX F
1 BIRT
2 PLAC birth place
2 DATE 1 JAN 1899
1 DEAT
2 PLAC death place
2 DATE 31 DEC 1990
1 FAMS @FAMILY@
0 @CHILD@ INDI
1 NAME /Child/
1 BIRT
2 PLAC birth place
2 DATE 31 JUL 1950
1 DEAT
2 PLAC death place
2 DATE 29 FEB 2000
1 FAMC @FAMILY@
0 @FAMILY@ FAM
1 MARR
2 PLAC marriage place
2 DATE 1 APR 1950
1 HUSB @FATHER@
1 WIFE @MOTHER@
1 CHIL @CHILD@
0 TRLR
view rawSimple.ged hosted with ❤ by GitHub
The following test was written to check that the importer was creating the correct model instance.

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
[TestMethod]
        public void TestImporter()
        {
            var storeId = Guid.NewGuid().ToString();
            GedcomImporter.Import(DataFolder + "simple.ged", "type=embedded;storesdirectory=c:\\brightstar;storename=" + storeId);
 
            var ctx = new GedComContext("type=embedded;storesdirectory=c:\\brightstar;storename=" + storeId);
 
            Assert.AreEqual(3, ctx.Individuals.Count());
            Assert.AreEqual(1, ctx.Families.Count());
 
            var family = ctx.Families.ToList()[0];
 
            Assert.IsNotNull(family.Husband);
            Assert.AreEqual("1 JAN 1899", family.Husband.BirthEvent.Date);
            Assert.AreEqual("M", family.Husband.Sex);
            Assert.AreEqual(family, family.Husband.SpouseFamilies().ToList()[0]);
 
            Assert.IsNotNull(family.Wife);
            Assert.AreEqual("1 JAN 1899", family.Wife.BirthEvent.Date);
            Assert.AreEqual("F", family.Wife.Sex);
            
            Assert.AreEqual(1, family.Children.Count());
 
            Assert.AreEqual("marriage place", family.MarriageEvent.Place);
        }
view rawGEDCOMImporterTest.cs hosted with ❤ by GitHub
One of the things about this project was that BrightstarDB persistence model allowed me to quickly define and evolve the data model as I went along. There was never an impedence mismatch between the data model and its storage.