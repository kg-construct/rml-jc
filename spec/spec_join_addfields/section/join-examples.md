## Join examples {#join-examples}

### Joining two CSV files
The following example illustrates a left join between two csv files. 

**chains.csv**
<pre class="ex-input">
"id","title","author"
"FNAC","The Republic","Plato"
"FNAC","Moby Dick","Melville"
"Waterstones","Poetics","Aristotle"
"Waterstones","Oliver Twist","Dickens"
</pre>
**bookstats.csv**
<pre class="ex-input">
"id","title","sales"
"book1","The Republic",12459
"book2","Moby Dick",335555
"book3","Poetics",254
</pre>

<pre class="ex-source">
<#bookstats> a rml:LogicalSource;
    rml:source [ 
        a rml:Source, a csvw:Table;
        csvw:url "/path/to/bookstats.csv";
    ];
    rml:referenceFormulation ql:CSV;
.
</pre>

<pre class="ex-source">
<#chains_with_sales> 
    rml:source [ 
        a rml:Source, a csvw:Table;
        csvw:url "/path/to/chains.csv"
        ];
    rml:referenceFormulation ql:CSV;
    rml:leftJoin [ 
        a rml:Join ;	 
        rml:parentLogicalSource <#bookstats> ;
        rml:joinCondition [
            rml:parent "title" ;
            rml:child "title" ;
        ] ;
        rml:addField "sales" ;
        rml:addFieldAlias [
            rml:parentField "id" ;
            rml:as "book_id" ;
        ] ;
    ] ;
.
</pre>
**Intermediate representation of the logical source <#chains_with_sales>**
<pre class="ex-intermediate">
| id          | title        | author    | sales  | book_id |
|-------------|--------------|-----------|--------|---------|
| FNAC        | The Republic | Plato     | 12459  | book1   |
| FNAC        | Moby Dick    | Melville  | 335555 | book2   |
| Waterstones | Poetics      | Aristotle | 254    | book3   |
| Waterstones | Oliver Twist | Dickens   |        |         |
</pre>

### Joining a json file with a csv file

The following examples illustrate a left join between a json file and a csv file.
The fields are added on the level of the iterator.

#### Example 1: the json file requires only one array in the path to the needed data 

**books.json**
<pre class="ex-input">
{
  "books": [
    { "title": "The Republic",
      "author": "Plato" },
    { "title": "Moby Dick",
      "author": "Melville" },
    { "title": "Poetics",
      "author": "Aristotle" },
    { "title": "Oliver Twist",
      "author": "Dickens" }
]
}
</pre>
**bookstats.csv**
<pre class="ex-input">
"id","title","sales"
"book1","The Republic",12459
"book2","Moby Dick",335555
"book3","Poetics",254
</pre>

<pre class="ex-source">
<#books_csv> a rml:LogicalSource;
    rml:source [ 
        a rml:Source, a csvw:Table;
        csvw:url "/path/to/bookstats.csv";
    ];
    rml:referenceFormulation ql:CSV;
.
</pre>

<pre class="ex-source">
<#books_with_sales> 
    rml:source [ 
        a rml:Source , dcat:Distribution ;
        dcat:accessURL <file:///path/to/chains.json> ;
    ] ;
    rml:referenceFormulation ql:JSONPath ;
    rml:iterator "$.books[*]" ;  
    rml:field [
        rml:name "name" ;
        rml:reference "$.name" ;
    ] ;
    rml:leftJoin [  
        a rml:Join ;
        rml:parentLogicalSource <#bookstats> ;
        rml:joinCondition [
            rml:parent "title" ;
            rml:child "title" ;
        ] ;
        rml:addField "sales" ;
        rml:addField "id" ;
    ] ;
.
</pre>

**Intermediate representation of the logical source <#books_with_sales>**
<pre class="ex-intermediate">
| iterator                                         | sales  | id      |
|--------------------------------------------------|--------|---------|
| { "title": "The Republic", "author": "Plato" }   | 12459  | book1   |
| { "title": "Moby Dick", "author": "Melville" }   | 335555 | book2   |
| { "title": "Poetics", "author": "Aristotle" }    | 254    | book3   |
| { "title": "Oliver Twist", "author": "Dickens" } |        |         |
</pre>


#### Example 2: the json file requires a second array in the path to the needed data

Because the json file acting as child logical source requires a second array in its json path to reach the needed data, it needs to be flattened using [Fields](). 

<aside class="issue">
Els: TODO add link to field spec
</aside>

**chains.json**
<pre class="ex-input">
{
  "chains": [
    { "name": "FNAC",
      "books": [
        { "title": "The Republic",
          "author": "Plato" },
        { "title": "Moby Dick",
          "author": "Melville" }]
    },
    { "name": "Waterstones",
      "books": [
        { "title": "Poetics",
          "author": "Aristotle" },
        { "title": "Oliver Twist",
          "author": "Dickens" }]
    }]
}
</pre>
**bookstats.csv**
<pre class="ex-input">
"id","title","sales"
"book1","The Republic",12459
"book2","Moby Dick",335555
"book3","Poetics",254
</pre>

<pre class="ex-source">
<#books_csv> a rml:LogicalSource;
    rml:source [ 
        a rml:Source, a csvw:Table;
        csvw:url "/path/to/bookstats.csv";
    ];
    rml:referenceFormulation ql:CSV;
.
</pre>

<pre class="ex-source">
<#chains_with_sales> 
    rml:source [ 
        a rml:Source , dcat:Distribution ;
        dcat:accessURL <file:///path/to/chains.json> ;
    ] ;
    rml:referenceFormulation ql:JSONPath ;
    rml:iterator "$.chains[*]" ;  
    rml:field [
        rml:name "name" ;
        rml:reference "$.name" ;
    ] ;
    rml:field [
        rml:name "book" ;
        rml:reference "$.books[*]" ;
        rml:field [
            rml:name "author" ;
            rml:reference "$.author" ;
        ] ;
        rml:field [
            rml:name "title" ;
            rml:reference "$.title" ;
        ] ;
    ] ;
    rml:leftJoin [  
        a rml:Join ;
        rml:parentLogicalSource <#bookstats> ;
        rml:joinCondition [
            rml:parent "title" ;
            rml:child "book.title" ;
        ] ;
        rml:addField "sales" ;
        rml:addFieldAlias [
            rml:parentField "id" ;
            rml:as "book_id" ;
        ] ;
    ] ;
.
</pre>
**Intermediate representation of the logical source <#chains_with_sales>**
<pre class="ex-intermediate">
| iterator | name        | book                                             | book.title   | book.author | sales  | book_id |
|----------|-------------|--------------------------------------------------|--------------|-------------|--------|---------|
| {...}    | FNAC        | { "title": "The Republic", "author": "Plato" }   | The Republic | Plato       | 12459  | book1   |
| {...}    | FNAC        | { "title": "Moby Dick", "author": "Melville" }   | Moby Dick    | Melville    | 335555 | book2   |
| {...}    | Waterstones | { "title": "Poetics", "author": "Aristotle" }    | Poetics      | Aristotle   | 254    | book3   |
| {...}    | Waterstones | { "title": "Oliver Twist", "author": "Dickens" } | Oliver Twist | Dickens     |        |         |
</pre>

### Joining a csv file with a xml file

The following example illustrates a inner join between a csv file and a xml file.


#### Example 1: the xml file requires only one array in the path to the needed data
Because of the inner join, the logical source <#chains_with_sales> contains one iteration less than the previous two examples.

<aside class="issue">
Els: TODO add link to field spec
</aside>

**chains.csv**
<pre class="ex-input">
"id","title","author"
"FNAC","The Republic","Plato"
"FNAC","Moby Dick","Melville"
"Waterstones","Poetics","Aristotle"
"Waterstones","Oliver Twist","Dickens"
</pre>

**bookstats.xml**
<pre class="ex-input">
&lt;bookstats&gt;
    &lt;book id="book1"&gt;
        &lt;title&gt;The Republic&lt;/title&gt;
        &lt;sales&gt;12459&lt;/sales&gt;
    &lt;/book&gt;
    &lt;book id="book2"&gt;
        &lt;title&gt;Oliver Twist&lt;/title&gt;
        &lt;sales&gt;89237472&lt;/sales&gt;
    &lt;/book&gt;
    &lt;book id="book3"&gt;
        &lt;title&gt;Moby Dick&lt;/title&gt;
        &lt;sales&gt;9092583&lt;/sales&gt;
    &lt;/book&gt;
&lt;/bookstats&gt;
</pre>

<pre class="ex-source">
<#bookstats>
    rml:source [ 
        a rml:Source , dcat:Distribution ;
        dcat:accessURL <file:///path/to/bookstats.xml> ;
    ] ;
    rml:referenceFormulation ql:XPath ;
    rml:iterator "/bookstats/book" ;
.
</pre>

<pre class="ex-source">
<#chains_with_sales> 
    rml:source [ 
        a rml:Source, a csvw:Table ;
        csvw:url "/path/to/chains.csv" ;
    ] ;
    rml:referenceFormulation ql:CSV;
    rml:leftJoin [ 
        a rml:Join ;	 
        rml:parentLogicalSource <#bookstats> ;
        rml:joinCondition [
            rml:child "title" ;
            rml:parent "title" ;
        ] ;
        rml:addField "sales" ;
        rml:addFieldAlias [
            rml:parentField "@id"
            rml:as "book_id"
        ]
    ] ;
.
</pre>
**Intermediate representation of the logical source <#chains_with_sales>**
<pre class="ex-intermediate">
| id          | title        | author    | sales  | book_id |
|-------------|--------------|-----------|--------|---------|
| FNAC        | The Republic | Plato     | 12459  | book1   |
| FNAC        | Moby Dick    | Melville  | 335555 | book2   |
| Waterstones | Poetics      | Aristotle | 254    | book3   |
</pre>

#### Example 2: the xml file requires a second array in the path to the needed data

Because XPATH can select the parent of a node, no flattening via fields is needed. 

**chains.csv**
<pre class="ex-input">
"id","title","author"
"FNAC","The Republic","Plato"
"FNAC","Moby Dick","Melville"
"Waterstones","Poetics","Aristotle"
"Waterstones","Oliver Twist","Dickens"
</pre>

**bookstats.xml**
<pre class="ex-input">
&lt;bookstats&gt;
    &lt;year id="2000"&gt;
        &lt;book id="book1"&gt;
            &lt;title&gt;The Republic&lt;/title&gt;
            &lt;sales&gt;22&lt;/sales&gt;
        &lt;/book&gt;
        &lt;book id="book2"&gt;
            &lt;title&gt;Oliver Twist&lt;/title&gt;
            &lt;sales&gt;48&lt;/sales&gt;
        &lt;/book&gt;
        &lt;book id="book3"&gt;
            &lt;title&gt;Moby Dick&lt;/title&gt;
            &lt;sales&gt;909&lt;/sales&gt;
        &lt;/book&gt;
    &lt;/year&gt;
    &lt;year id="2001"&gt;
        &lt;book id="book1"&gt;
            &lt;title&gt;The Republic&lt;/title&gt;
            &lt;sales&gt;78&lt;/sales&gt;
        &lt;/book&gt;
        &lt;book id="book2"&gt;
            &lt;title&gt;Oliver Twist&lt;/title&gt;
            &lt;sales&gt;795&lt;/sales&gt;
        &lt;/book&gt;
    &lt;/year&gt;
&lt;/bookstats&gt;
</pre>

<pre class="ex-source">
<#bookstats>
    rml:source [ 
        a rml:Source , dcat:Distribution ;
        dcat:accessURL <file:///path/to/bookstats.xml> ;
    ] ;
    rml:referenceFormulation ql:XPath ;
    rml:iterator "/bookstats/year/book" ;
    ] ;
.
</pre>

<pre class="ex-source">
<#chains_with_sales> 
    rml:source [ 
        a rml:Source, a csvw:Table ;
        csvw:url "/path/to/chains.csv" ;
    ] ;
    rml:referenceFormulation ql:CSV;
    rml:leftJoin [ 
        a rml:Join ;	 
        rml:parentLogicalSource <#bookstats> ;
        rml:joinCondition [
            rml:child "title" ;
            rml:parent "title" ;
        ] ;
        rml:addFieldAlias [
            rml:parentField "parent::year/@id"
            rml:as "year"
        ] ;
        rml:addFieldAlias [
            rml:parentField "sales"
            rml:as "sales"
        ] ;
        rml:addFieldAlias [
            rml:parentField "@id"
            rml:as "book_id"
        ] ;
    ] ;
.
</pre>
**Intermediate representation of the logical source <#chains_with_sales>**
<pre class="ex-intermediate">
| id          | title        | author    | year | sales | book_id |
|-------------|--------------|-----------|------|-------|---------|
| FNAC        | The Republic | Plato     | 2000 | 22    | book1   |
| FNAC        | Moby Dick    | Melville  | 2000 | 48    | book2   |
| Waterstones | Poetics      | Aristotle | 2000 | 909   | book3   |
| FNAC        | The Republic | Plato     | 2001 | 78    | book1   |
| FNAC        | Moby Dick    | Melville  | 2001 | 795   | book2   |
</pre>