## MOTIVATION

### FIELDS
R2MRL is limited to data from a single SQL database
RML maps data from heterogeneous problems, also nested files such as json and xml. 

Nested files come with specific problems. 
Sometimes a second array needs to be traversed to reach the wanted data. This requires an iterator in an iterator. 
This limits the use of nested files in RML today.   
  
**Fields** offer a solution to overcome this problem: fields flatten the file by allowing a iterator as part of a (nested) field.
Fields are needed if we want to have the same mapping possibilities for nested data as we have for tabular data. 

### JOINS IN LOGICAL SOURCES
R2RML allows queries over tables / views as logical source for triples maps.
This allows:
- the use of a value from a join as literal
- the use of a  value from a join in subject, predicate or graph
- the combination of data from 2 sources in one term
- the combination of data from 3+ sources

RML has no alternative for this today, except for views over one relational database.
Query was present since the beginning in RML but cannot be used over heterogeneous sources. 

The proposal to allow joins in logical sources want to add the same expressiveness to RML as R2RML has.   

This proposal runs into the same issue with nested files that is already present in RML today: 
as soon as second array needs to be traversed, **fields** are needed to reach the data. 
This is also a problem in RML for all references and the existing joins through a referencing object map. 

Therefore, the proposal to allow **joins in logical sources** comes with the same limitations, as long as fields are not introduced. 
Nevertheless, it already extends the expressiveness of RML for many, less complex, cases. 

### In RML SCOPE? 

For me both problems are in the scope of RML, because RML aims to map data from heterogeneous sources
and should therefore look for solutions related to this heterogeneity.  

Otherwise, we can just trust other tools to convert all data to tabular data, which is put into an SQL database. 
In that case RML is not needed anymore, and we can stick to R2RML. 

, and put into an SQL database
****
I know nothing about recursion, but will read the articles which Maria-Esther has advised. 
I also welcome your examples of recursions, so I can start to understand the issue which you have raised.  


