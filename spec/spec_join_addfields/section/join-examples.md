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
    rml:referenceFormulation rml:CSV;
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

The following example illustrates a left join between a json file and a csv file. 
The fields are added on the level of the iterator. 
Because the json file acting as child logical source is a nested file, it needs to be flattened using [Fields](). 
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
    rml:referenceFormulation rml:CSV;
.
</pre>

<pre class="ex-source">
<#chains_with_sales> 
    rml:source [ 
        a rml:Source , dcat:Distribution ;
        dcat:accessURL <file:///path/to/chains.json> ;
    ] ;
    rml:referenceFormulation rml:JSONPath ;
    rml:iterator "$.chains[*]" ;  
    rml:field [
        rml:name "name" ;
        rml:reference "$.name" ;
    ] ;
    rml:field [
        rml:name "book" ;
        rml:reference "$.books[*]" ;
    ] ;
    rml:field [
        rml:name "author" ;
        rml:reference "$.author" ;
    ] ;
    rml:field [
        rml:name "title" ;
        rml:reference "$.title" ;
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
Because the xml file acting as parent logical source is a nested file, it needs to be flattened using [Fields]().
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
&lt;root&gt;
    &lt;bookStats&gt;
        &lt;element&gt;
            &lt;id&gt;book1&lt;/id&gt;
            &lt;book&gt;The Republic&lt;/book&gt;
            &lt;sales&gt;12459&lt;/sales&gt;
        &lt;/element&gt;
        &lt;element&gt;
            &lt;id&gt;book2&lt;/id&gt;
            &lt;book&gt;Oliver Twist&lt;/book&gt;
            &lt;sales&gt;89237472&lt;/sales&gt;
        &lt;/element&gt;
        &lt;element&gt;
            &lt;id&gt;book3&lt;/id&gt;
            &lt;book&gt;Moby Dick&lt;/book&gt;
            &lt;sales&gt;9092583&lt;/sales&gt;
        &lt;/element&gt;
    &lt;/bookStats&gt;
&lt;/root&gt;
</pre>

<pre class="ex-source">
<#bookstats>
    rml:source [ 
        a rml:Source , dcat:Distribution ;
        dcat:accessURL <file:///path/to/bookstats.xml> ;
    ] ;
    rml:referenceFormulation rml:XPath ;
    rml:iterator "/root/bookStats/element" ;
    rml:field [
        rml:name "book_id" ;
        rml:reference "/element/id" ;
    ] ;
    rml:field [
        rml:name "book" ;
        rml:reference "/element/book" ;
    ] ;
    rml:field [
        rml:name "sales" ;
        rml:reference "/element/sales ;
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
            rml:parent "book" ;
        ] ;
        rml:addField "sales" ;
        rml:addField "book_id"
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


