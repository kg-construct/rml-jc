# JOIN RELATED PROPOSALS AND ISSUES

In this document we are collecting existing join related proposals and issues. 

Contributors (until now): Ben De Meester, Lionel Tailhardat, Els de Vleeschauwer.  
Personal views/summaries/comments are preceeded by the name of the contributor.   

## Proposals / existing constructions

### [Referencing term maps](https://github.com/kg-construct/rml-core/blob/main/spec/docs/joins.md)  

#### *Els*
Solution for reuse of term maps, defines the term on one place and refer to it, changes will be automatically aligned.
When the logical sources of the child and parent are identical, this can be done without conditions. Otherwise at least one condition is needed. 
The conditions used can be either all **reference conditions** (if all needed info is available in the child’s logical source) or all **join conditions** (if it is needed to make a connection between the child’s logical source and the parent’s logical source).


### [Reference conditions](https://kg-construct.github.io/workshop/2023/resources/paper11.pdf)

#### *Els*
To reuse a term map without using the parent’s logical source.  
Effect: to reuse the term map of the parent, you need to replace all references in that term map by the corresponding references as defined in the reference conditions. An error should be thrown if not all parent references can be replaced by child references (i.e. when reference conditions are missing).   
Reason: unless confirmed by the reference conditions, we cannot know if a reference in the child’s logical source has the same semantic ‘meaning’ as a reference with the same ‘name’ in the parent’s logical source. If the term map of the parent is changed over time, including a new reference, this  change is automatically applied to the child. If the new reference of the parent’s logical source occurs also in the child’s logical source, it will be used, without the ‘permission/verification’ of the developer of the mapping file. This can cause unnoticed errors.   
Referential integrity inside the resulting knowledge graph is not secured, when using reference conditions.  

### Join conditions 
(not a proposal of course, but included to explain the difference with reference conditions)  
#### *Els*
To reuse a term map by using data from both the child’s logical source and the parent’s logical source.   
Effect: the term map generates terms using the references of the parent’s logical sources, and those will  be used by the child only if a match is found in the child’s logical source after applying the join conditions.   
No error is needed if the parent term map uses a reference which is not appearing in the join conditions, as the child’s logical source is not used to generate the terms; the join conditions are only used to connect the parent’s term to the other terms of the child’s triple.  
Join conditions in combination with **referencing term maps** are a shortcut for **RML views**, handling only the joining of 2 sources. This construction secures referential integrity in the resulting knowledge graph.



### [RML Views](https://2023.eswc-conferences.org/wp-content/uploads/2023/05/paper_Arenas-Guerrero_2023_Boosting.pdf)

#### *Els*
Handles transformations and joins of sources on logical source level by using SQL queries and SQL functions. Results in a new logical source.    
It would be good to also propose an RML formulation equivalent to the SQL formulation, so SQL knowledge is not required and the views can be formulated as well in RML language. e.g. using joins, nested joins, making virtual references using function term maps and renaming them with an alias (with **RML fields**?, should be reusable immediately as a reference in a template).  
Maybe SQL formulation can be automatically converted to RML formulations? This gives the advantage of a compact and known formulation for developers acquainted with relation databases, and makes if easier to push join and transformation operations back to source systems or to reuse existing libraries.   
Can be used to replace the classic **join conditions** in combination **referencing object maps**, and is also a good solution for using data of other sources as literal in an object term map:
[https://github.com/kg-construct/rml-core/issues/62](https://github.com/kg-construct/rml-core/issues/62)
Once RML Views are accepted, I would personal not need for referencing term maps. I think that the existing referencing object maps are sufficient in that case, just to express relations between triples maps (and in the resulting knowledge graph). 

### [RML Fields](https://biblio.ugent.be/publication/8720875)

#### *Els*
RML Fields was a proposal to integrate nested data. Maybe this structure can be extended to formalise virtual (or calculated) references / aliases in general, enabling the renaming of references after a join operation on logical source level, and references for values transformed by fno functions. The reference can then be used inside an rr:template or a rr:reference by more than one term map. This can also be a solution for the remark of static rr:templates (see issue below)

#### *Lionel*
I tried it with the mapping-challenges dataset and Orange internal data, but no success because of missing implementation of the feature in RMLMapper => https://github.com/kg-construct/mapping-challenges/issues/40 

### [Patching queries and URI patterns checking](https://w3id.org/kg-construct/workshop/2023/resources/paper3.pdf)
#### *Lionel*
Proposal: automated patching generation can be enabled by browsing RML files for `rr:predicateObjectMap [rr:objectMap [rml:reference "<someRef>"]]`, potentially with an additional `toPatchWith(<someGraphPattern>)` property for better end-to-end process automation and automated URI template checking.  
Patching is defined by applying SPARQL Update queries (e.g. of the DELETE & INSERT form) at the knowledge graph level posterior to running several RML mapping & data ingest processes.  
Patching query example:  
```
DELETE { GRAPH ?gs { ?s noria:logOriginatingManagedObject ?o .  } }
INSERT { GRAPH ?gs { ?s noria:logOriginatingManagedObject ?st . } }
WHERE { 
    GRAPH ?gs {
   	 ?s a noria:EventRecord; noria:logOriginatingManagedObject  ?o .
   	 FILTER isLiteral(?o) . }
    GRAPH ?gt {
   	 ?st a noria:Resource; noria:resourceHostName ?ot . }
    FILTER (?o = ?ot) .
}
```

The patching approach is complementary to using joins and crafted URIs, notably overcoming issues related to:
- joins when data sources used in joins are not present at the same time (which might be the case for stream processing or KGC flows with a great deal of sources) or when the execution time of joins is excessive,  
- crafted URIs when rr:subjectMap with rr:templates are inconsistent across RML rule set. Options here could be 1) check mappings against a set of URI patterns, or 2) use variables to build URIs in an error prone way at the mapping stage.    
Challenge with the patching approach is two fold:  
- Designing the patching queries,  
- Ensure consistency between mapping rules and patching queries, thereby facilitating the maintenance of both aspects.    
Options for the patching approach, in relation to the introductory “proposal” above (non exhaustive):  
- Deduce the patching from the RML files using additional RML predicates akin to **reference conditions** (see Els’ proposal above).
- Relate `rr:TriplesMap` to the relevant pathing queries or patching templates using additional RML predicates such as `toPatchWith(<someGraphPattern>)`.  
The current approach used in the “Designing NORIA” paper is to generate patching queries and relate queries to rr:TriplesMap using a specification file with definitions such as (YAML syntax):
```
- id: MP_Solaire_Resource_002
  ExpectedPattern:
	Source: '?s a noria:Resource; noria:resourceManagedBy ?o .'
	Target: '?st a org:OrganizationalUnit; org:identifier ?ot .'
  Remarks:
	- TriplesMap: MP_Solaire_Resource
  QueryGeneratorTemplate:
	- extract-query-literal.sparql
	- link-query-literal.sparql
  SolvedBy:
	- extract-query-literal-MP_Solaire_Resource_002.sparql
	- link-query-literal-MP_Solaire_Resource_002.sparql
```
- URI patterns checking assumes prior knowledge of the URI patterns of the knowledge graph. The approach tackles the need to check that the resulting knowledge graph of a complex KGC process (i.e. after inserting triples coming from many mapping processes) is coherent with the URI patterns.  
URI patterns example in YAML syntax:  
```
- Base: https://w3id.org/noria/graph/
- Group: document
  Usage: Document entities related to Incident Management and Change Management
  Class:
	- Name: 'noria:TroubleTicket'
	- Name: 'noria:TroubleTicketNote'
	- Name: 'noria:ChangeRequest'
	- Name: 'noria:DocumentAttachment'
- Group: location
  Usage: Geographical entities
  Class:
	- Name: 'bot:Site'
	- Name: 'bot:Building'
	- Name: 'bot:Storey'
	- Name: 'bot:Space'
	- Name: 'noria:Room'
	- Name: 'noria:Locus'
```
References and remarks towards automated checking:
- “Extracting URI Patterns from SPARQL Endpoints”, Alessandro Adamou (2014).
- SHACL does not provide a template definition for URIs, but it allows stating that an object is of type URI.


### [Dynamic windows](https://arxiv.org/abs/2210.14599)

#### *Els*
Solution for joining data streams

### [Iteration and reference identifiers](https://github.com/kg-construct/mapping-challenges/issues/43)

#### *Ben*

To join based on something else than values


## *Ben*: Known issues around joins

- Joining multiple nodes: 
    - <https://github.com/RMLio/rmlmapper-java/issues/80>
- Joining within the same logical source: 
    - <https://github.com/RMLio/rmlmapper-java/issues/199>
- Joining based on hierarchy: 
    - <https://github.com/kg-construct/rml-questions/discussions/19#discussioncomment-3136747>
    - <https://github.com/kg-construct/mapping-challenges/tree/main/challenges/access-fields-outside-iteration>
- Join condition based on transformed data (e.g. fuzzy matching)
- Join condition based on constant data: 
    - <https://github.com/RMLio/yarrrml-parser/issues/39>
- Join to get literal value instead of subject of triples map: 
    - <https://github.com/kg-construct/mapping-challenges/tree/main/challenges/join-on-literal> 
    - <https://github.com/kg-construct/rml-core/issues/62>
- Join streaming data needs windows 

## Other issues
 
### *Lionel*: static rr:templates 

rr:template are static (in the sense that they cannot be manipulated through FNOs), which prevents advanced processing useful to both joins and producing cleaner knowledge graphs (e.g. using a slugify function).
