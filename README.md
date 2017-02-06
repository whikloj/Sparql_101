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

## Sparql

The W3C Spec on Sparql 1.1 is split up in to a bunch of different specifications. We'll concentrate on the [Query](https://www.w3.org/TR/sparql11-query/) and maybe some [Update](https://www.w3.org/TR/sparql11-update/).

The 4 forms of Sparql querying are **SELECT**, **CONSTRUCT**, **ASK** and **DESCRIBE**

We'll use a larger set of Flintstones data for these examples.

| Subject | Predicate | Object |
| --- |:---:|---:|
| :Fred | :hasAge | 25 |
| :Fred | :hasSpouse | :Wilma |
| :Fred | :livesIn | :Bedrock |
| :Wilma | :livesIn | :Bedrock |
| :Barney | :hasSpouse | :Betty |
| :Barney | :livesIn | :Bedrock |
| :Betty | :livesIn | :Bedrock |
| :Fred | :hasChild | :Pebbles |
| :Barney | :hasChild | :Bamm-Bamm |
| :Pebbles | :hasSpouse | :Bamm-Bamm |
| :Pebbles | :hasChild | :Roxy |
| :Pebbles | :hasChild | :Chip |

### Select

This is the form most people are comfortable with. You bind values to variables and return those variables.

An example using the above would be to find all of Pebbles children
```
SELECT ?kids WHERE {
   :Pebbles :hasChild ?kids .
}
```




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
