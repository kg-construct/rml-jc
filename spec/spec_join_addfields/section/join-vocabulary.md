## Join vocabulary {#join-vocabulary}

The Join vocabulary namespace is http://w3id.org/rml/
and it's prefix is `rml`.

### Joins

A <dfn>Join</dfn> (`rml:Join`) is an operation that extends one logical source (the child logical source) with data from another logical source (the parent logical source).

A [=Join=] (`rml:Join`) MUST contain:
- exactly one [=parent logical source=] property (`rml:parentLogicalSource`) to describe the data source that supplies the additional data. 
- at least one [=join condition=] property (`rml:joinCondition`) to describe which values are compared to join the two data sources.
- at least one [=add field=] property (`rml:addField`) or one [=add field alias=] property (`rml:addFieldAlias`) to describe a field from the parent logical source that is added to the child logical source. 

| Property                  | Domain     | Range               |
|---------------------------|------------|---------------------|
| `rml:parentLogicalSource` | `rml:Join` | `rml:LogicalSource` |
| `rml:joinCondition`       | `rml:Join` | `rml:JoinCondition` |
| `rml:addField`            | `rml:Join` | `Literal`           |
| `rml:addFieldAlias`       | `rml:Join` | `rml:FieldAlias`    | 

<aside class="issue">
Els: in the RML CORE spec rml:Join is the class containing the join conditions. Can this still be adapted?
</aside>

### Parent Logical Source

The Range of a <dfn>parent logical source</dfn> property is a [Logical Source](https://kg-construct.github.io/rml-io/spec/docs/#defining-logical-sources) and describes the data source that supplies the additional data and is the right side of the join. 

### Join Conditions

A <dfn>Join Condition</dfn> is represented by a resource that MUST contain exactly one value for each of the following two properties:

- a <dfn>child map</dfn> (`rml:childMap`), whose value is an [Expression Map](https://kg-construct.github.io/rml-core/spec/docs/#expression-map-rml-expressionmap) (`rml:ExpressionMap`), 
which MUST include references that exists in the child logical source, or it should have a constant value.

- a <dfn>parent map</dfn> (`rml:parentMap`), whose value is an [Expression Map](https://kg-construct.github.io/rml-core/spec/docs/#expression-map-rml-expressionmap) (`rml:ExpressionMap`),
which, as the join condition's parent map, MUST include references that exist in the logical source specified by the [=parent logical source=] property or it should have a constant value.

The join condition returns true when values produced by the child map and the parent map during the iteration are equal. 
<aside class="note">
String values are compared.
Data types are not taken into account. 
`1.0` will not match with `1.00`. 
To secure this match a transformation with <a href="https://kg-construct.github.io/rml-fnml/ontology/documentation/index-en.html">RML-FNML:Functions</a> needs to be configured. 
</aside>

<aside class="note">
This definition should be in line with the definition in RML CORE, with one small difference: it refers directly to a parent logicial source, and not to the logical source of the parent triples map.
</aside>

| Property                    | Domain               | Range                     |
| --------------------------- | -------------------- | ------------------------- |
| `rml:childMap`              | `rml:JoinCondition`  | `rml:ExpressionMap`       |
| `rml:parentMap`             | `rml:JoinCondition`  | `rml:ExpressionMap`       |

#### Shortcuts

If the value of the [=child map=] property (`rml:childMap`) is a [reference-valued Expression Map](https://kg-construct.github.io/rml-core/spec/docs/#reference-rml-reference),
then the `rml:child` shortcut could be used.

Similarly, if value of the [=parent map=] (`rml:parentMap`) is a [reference-valued Expression Map](https://kg-construct.github.io/rml-core/spec/docs/#reference-rml-reference),
then the `rml:parent` shortcut could be used.

| Property                    | Domain               | Range                     |
| --------------------------- | -------------------- | ------------------------- |
| `rml:child`                 | `rml:JoinCondition`  | `Literal`                 |
| `rml:parent`                | `rml:JoinCondition`  | `Literal`                 |

<aside class="issue">
Els: or can we just refer to rml core and not specify any definitions here?
</aside>
<aside class="issue">
Els: can we also optionally declare a join function here, to allow not only equijoins (default) but also other joins
</aside>

### Add Field 

The value of an <dfn>add field</dfn> property is a [reference expression](https://kg-construct.github.io/rml-core/spec/docs/#dfn-reference-expression), that is valid in the parent logical source,
and specifies which field of the parent logical source is added to the child logical source. 

### Add Field Alias 

If the reference expression of the field of the parent logical source has a duplicated value in the child logical source, the **add field alias** property (`rml:addFieldAlias`) MUST be used instead of the **add field** property. 
The value of an <dfn>add field alias</dfn> property (`rml:addFieldAlias`) is an object that MUST contain: 
- exactly one **parent field** property (`rml:parentField`), whose value is a [reference expression](https://kg-construct.github.io/rml-core/spec/docs/#dfn-reference-expression), that is valid in the parent logical source,
  and specifies which field of the parent logical source is added to the child logical source.
- exactly one **as** property (`rml:as`) to specify an alias for the added field.

| Property          | Domain               | Range     |
|-------------------|----------------------|-----------|
| `rml:parentField` | `rml:AddFieldAlias ` | `Literal` |
| `rml:as`          | `rml:AddFieldAlias`  | `Literal` |

<aside class="issue">
Els: can we use the Field terminology here, `rml:reference` instead of `rml:parentField` and `rml:name` instead of `rml:as`. 
I read in the Field spec that a field gives a name to a reference, and that is exactly what is happening here as well. 
</aside>

### Join types {#dfn-join-type}

The Logical Source vocabulary is extended with join properties, specifying the join type, i.e. a [=left join=] and a [=inner join=].

A <dfn>left join</dfn> (`rml:leftJoin`) is the equivalent of a left outer join in SQL, where the child logical source is the left part of the join, and the parent logical source is the right part of the join.  
A <dfn>inner join</dfn> (`rml:innerJoin`) is the equivalent of an inner join in SQL.   

<aside class="issue">
Els: Should we refer to the SQL definitions? Or should we explain further what is a left outer join and what is an inner join. Or should we add the JOINT SQL query as is done in R2RML
</aside>
<aside class="issue">
Els: Should we open an option to add more join types in future? 
</aside>

A Logical Source MAY have at most one join property.

| Property        | Domain              | Range               |
|-----------------|---------------------|---------------------|
| `rml:leftJoin`  | `rml:LogicalSource` | `rml:Join`          |
| `rml:innerJoin` | `rml:LogicalSource` | `rml:Join`          |

Under discussion: the alternative is to work with subclasses of rml:Join.
By default the Join is considered a left join (`rml:LeftJoin`), if not specified.
The join type may be altered by using a subclass, e.g. a inner join (`rml:InnerJoin`)
