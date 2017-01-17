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
