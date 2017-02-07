# Querying and SPARQL update

This repository covers the content of the above titled workshop at IslandoraCon 2017.

## Outline
* Introduction
* RDF and SPARQL
* Fedora Resource Index
* Sparql

## Introduction

Fedora (version 3) provided a built-in semantic store to hold the information about Fedora resources. This is commonly referred to as the [Resource Index](https://wiki.duraspace.org/display/FEDORA38/Resource+Index)

## RDF, Triplestores and SPARQL, oh my!

The RELS-EXT datastream on a Fedora resource contains an [RDF](https://www.w3.org/RDF/) description of the relationships between that resource and others, it also contains other information about that resource. 

Those statements are also added to the **Resource Index**, in the case of a standard Islandora and Fedora installation this is [Mulgara](http://www.mulgara.org/).

Mulgara is a *triplestore* or an RDF Store. Its called a *triplestore* because it *stores triples*.

What's a triple?
> A triple is the atomic unit of data in the semantic web; just like a row was the atomic unit of data in the old world, the triple is the row of the new world.
-- https://data-gov.tw.rpi.edu/wiki/Triples

A triple is a unit of data that has the form of *SUBJECT PREDICATE OBJECT*

The **Subject** is what the statement is about, the **Predicate** is the *property* or *data-type* and the **Object** is the value.

The [Flintstones example by OntoText](http://graphdb.ontotext.com/devhub/rdfs.html)

The full graph from the above example could be

| Subject | Predicate | Object |
| --- |:---:|---:|
| :Fred | :hasAge | 25 |
| :Fred | :hasSpouse | :Wilma |
| :Fred | :livesIn | :Bedrock |



So as an example a photo of [Louis Riel](https://www.collectionscanada.gc.ca/confederation/023001-4000.61-e.html) from the UM DigitalCollections site contains the following RELS-EXT
```
<rdf:RDF>
  <rdf:Description rdf:about="info:fedora/uofm:9333">
    <fedora:isMemberOfCollection rdf:resource="info:fedora/uofm:riel"/>
    <fedora-model:hasModel rdf:resource="info:fedora/uofm:highResImage"/>
    <fedora-model:hasModel rdf:resource="info:fedora/islandora:compoundCModel"/>
    <islandora:isManageableByUser>xxxx</islandora:isManageableByUser>
    <islandora:isManageableByRole>yyyy</islandora:isManageableByRole>
  </rdf:Description>
</rdf:RDF>
```
So your Resource index will have the following triples.

| Subject | Predicate | Object |
| --- |:---:|:---:|
| info:fedora/uofm:9333 | fedora:isMemberOfCollection | info:fedora/uofm:riel |
| info:fedora/uofm:9333 | fedora-model:hasModel | info:fedora/islandora:compoundCModel |
| info:fedora/uofm:9333 | fedora-model:hasModel | info:fedora/uofm:highResImage |
| info:fedora/uofm:9333 | islandora:isManageableByUser | xxxx |
| info:fedora/uofm:9333 | islandora:isManageableByRole | yyyy |

### RDF and shared ontologies

The benefit of RDF is that one can share their ontology with the world or borrow from (and build upon) another's work.

You can go it alone and create your own ontology which is allowed. For our Flintstones we might have [this](Flintstones.ttl). This ontology defines the relationships between people and between people and places.

But there are others that will have objects that share these relationships and you can benefit from sharing those relationships. For instance the [FOAF (Friend Of A Friend) ontology](http://xmlns.com/foaf/spec/) "... is a project devoted to linking people and information using the Web."

Because our Bedrock family members are all "people" and foaf has a class for that
> The Person class represents people. Something is a Person if it is a person. We don't nitpic about whether they're alive, dead, real, or imaginary. 

We can extend our data by defining each entry as a **foaf:Person**. So now we have [this](Flintstones_2.ttl).

Not bad, but we have used our own vocabulary to define the _spouseOf_ and _parentOf_ relationships. I'm sure someone else somewhere has defined those relationships.

The [**relationship**](http://vocab.org/relationship/) ontology also does this. So we don't have to re-invent the wheel and can make use of their _http://purl.org/vocab/relationship/spouseOf_ and _http://purl.org/vocab/relationship/parentOf_ predicates.

Now our data becomes [this](Flintstones_3.ttl).

### Sparql

The W3C Spec on Sparql 1.1 is split up in to a bunch of different specifications. We'll concentrate on the [Query](https://www.w3.org/TR/sparql11-query/) and maybe some [Update](https://www.w3.org/TR/sparql11-update/).

The 4 forms of Sparql querying are **SELECT**, **CONSTRUCT**, **ASK** and **DESCRIBE**

We'll use our last set of data from above for these examples.

#### Select

This is the form most people are comfortable with. You bind values to variables and return those variables.

An example using the above would be to find all of Pebbles children
```
PREFIX ff: <test:flintstones#> 
PREFIX vocab: <http://purl.org/vocab/relationship/>
SELECT ?kids WHERE {
   ff:Pebbles vocab:parentOf ?kids .
}
```
This will return

| ?kids |
| :---: |
| &lt;test:flintstones#Chip&gt; |
| &lt;test:flintstones#Roxy &gt; |

We know that the _vocab:parentOf_ predicate defines a parent to child relationship, this means that the parent is ALWAYS the subject and the child is ALWAYS the object. This makes it easy to determine parentage by slightly adjusting our query.

```
PREFIX ff: <test:flintstones#> 
PREFIX vocab: <http://purl.org/vocab/relationship/>
SELECT ?parents WHERE {
   ?parents vocab:parentOf ff:Chip .
}
```

| ?parents |
| :---: |
| &lt;test:flintstones#Bamm-Bamm&gt; |
| &lt;test:flintstones#Pebbles&gt; |

What about following the relationships further?
```
PREFIX ff: <test:flintstones#> 
PREFIX vocab: <http://purl.org/vocab/relationship/>
SELECT ?grandChild WHERE {
   ff:Fred vocab:parentOf ?kid .
   ?kid vocab:parentOf ?grandChild .
}
```

So here we found all the children of Fred and assigned them to a variable, then we use the values in that variable and find all children of those people. This becomes two generations and allows us to find grandchildren.

| ?grandChild |
| :---: |
| &lt;test:flintstones#Chip&gt; |
| &lt;test:flintstones#Roxy&gt; |

#### Construct

Construct returns a single RDF graph generated using a template and replacing variables with values in other graphs. 

For instance what if instead of just finding out who Fred's grandchildren are we want to make a relationship for that.
```
PREFIX ff: <test:flintstones#> 
PREFIX vocab: <http://purl.org/vocab/relationship/>

CONSTRUCT {
  ?grandparent ff:grandparentOf ?grandchild
} WHERE {
  ?grandparent vocab:parentOf ?kid .
   ?kid vocab:parentOf ?grandchild .
}
```

| subject | predicate | object |
| :---: | :---: | :---: |
| &lt;test:flintstones#Barney&gt; | &lt;test:flintstones#grandparentOf&gt; | &lt;test:flintstones#Chip&gt; | 
| &lt;test:flintstones#Betty&gt; | &lt;test:flintstones#grandparentOf&gt; | &lt;test:flintstones#Chip&gt; | 
| &lt;test:flintstones#Barney&gt; | &lt;test:flintstones#grandparentOf&gt; | &lt;test:flintstones#Roxy&gt; | 
| &lt;test:flintstones#Betty&gt; | &lt;test:flintstones#grandparentOf&gt; | &lt;test:flintstones#Roxy&gt; | 
| &lt;test:flintstones#Fred&gt; | &lt;test:flintstones#grandparentOf&gt; | &lt;test:flintstones#Chip&gt; | 
| &lt;test:flintstones#Wilma&gt; | &lt;test:flintstones#grandparentOf&gt; | &lt;test:flintstones#Chip&gt; | 
| &lt;test:flintstones#Fred&gt; | &lt;test:flintstones#grandparentOf&gt; | &lt;test:flintstones#Roxy&gt; | 
| &lt;test:flintstones#Wilma&gt; | &lt;test:flintstones#grandparentOf&gt; | &lt;test:flintstones#Roxy&gt; | 


### Predicates and Prefixes

One of the things to remember when writing any Sparql queries is to always define your prefixes. 

We've become spoiled because Fedora has a set of prefixes that are recognized by default. These are:

| Prefix | Namespace |
| --- | --- |
| test	 | http://example.org/terms# |
| fedora-rels-ext |	info:fedora/fedora-system:def/relations-external# |
| fedora | info:fedora/ |
| rdf | http://www.w3.org/1999/02/22-rdf-syntax-ns# |
| fedora-model | info:fedora/fedora-system:def/model# |
| fedora-view | info:fedora/fedora-system:def/view# |
| mulgara | http://mulgara.org/mulgara# |
| dc | http://purl.org/dc/elements/1.1/ |
| xml-schema | http://www.w3.org/2001/XMLSchema# |

So many of our Islandora queries appear in code [like this](https://github.com/Islandora/islandora_paged_content/blob/7.x/includes/utilities.inc#L61-L76).

The internally registered prefixes allow you to avoid registering them in your query. However, because this is only good for Mulgara you run into trouble if you switch to using a different triplestore.

You can avoid that problem by simply registering each prefix you intend to use in your query, so the above example becomes.
```
  PREFIX fedora-rels-ext: <info:fedora/fedora-system:def/relations-external#>
  PREFIX fedora-model: <info:fedora/fedora-system:def/model#>
  PREFIX fedora-view: <info:fedora/fedora-system:def/view#>
  PREFIX islandora-rels-ext: <http://islandora.ca/ontology/relsext#>
  SELECT ?pid ?page ?label ?width ?height
  FROM <#ri>
  WHERE {
    ?pid fedora-rels-ext:isMemberOf <info:fedora/{$object->id}> ;
         fedora-model:label ?label ;
         islandora-rels-ext:isSequenceNumber ?page ;
         fedora-model:state fedora-model:Active .
    OPTIONAL {
      ?pid fedora-view:disseminates ?dss .
      ?dss fedora-view:disseminationType <info:fedora/*/JP2> ;
           islandora-rels-ext:width ?width ;
           islandora-rels-ext:height ?height .
   }
  }
  ORDER BY ?page
```

Now this query will operate identically on any triplestore that support Sparql.
