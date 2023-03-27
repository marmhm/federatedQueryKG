In this use case, participants are requested to efficiently re-write and execute the query listed below, which uses many data sources, in federated manner.

# Using WorldCat Data to Feed a Wikidata Query

A simple use case, in which We load the graph for a given OCLC record URI, extracting the predicate for the OCLC Number and then querying Wikidata's SPARQL endpoint based on that number.

Datasets:
* WorldCat Bib: graph to host on GraphDB at https://graphdb.dumontierlab.com
* Wikidata: hosted on Virtuoso at https://query.wikidata.org/sparql

```SPARQL
PREFIX schema: <http://schema.org/>
PREFIX library: <http://purl.org/library/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT ?work
FROM <http://www.worldcat.org/oclc/1210>
WHERE { 
<http://www.worldcat.org/oclc/1210> library:oclcnum ?oclcNumber. 

SERVICE <https://query.wikidata.org/sparql> { 
?work wdt:P243 ?oclcNumber. }
}
```

# Querying Multiple Datasets

Find all the "Works" Wikidata and the British Library think are associated with the creator within a specific WorldCat Bib graph and returning their URIs and their "English" title.

Datasets:
* WorldCat Bib: graph to host on GraphDB at https://graphdb.dumontierlab.com
* Wikidata: hosted on Virtuoso at https://query.wikidata.org/sparql
* British library catalogue dataset: graph to host on GraphDB at https://graphdb.dumontierlab.com

```SPARQL
PREFIX schema: <http://schema.org/> 
PREFIX library: <http://purl.org/library/> 
PREFIX wdt: <http://www.wikidata.org/prop/direct/> 
PREFIX dct: <http://purl.org/dc/terms/>
 
SELECT ?work ?workLabel
FROM <http://www.worldcat.org/oclc/470488115> 
WHERE {  
<http://www.worldcat.org/oclc/470488115> schema:author ?creatorURI.
BIND(replace(STR(?creatorURI), "^(.*[\\/])*", "") AS ?creatorID)  
 
{SERVICE <https://query.wikidata.org/sparql> {
  ?author wdt:P214 ?creatorID.
  ?work wdt:P50 ?author .
  ?work wdt:P1476 ?workLabel
  } 
}  
UNION
{
{  
?work dct:creator ?bl_creator.
?work dct:title ?workLabel
 }
} 
}
```

# Creating a New Graph from Data in Different Datasets

Get multi-lingual title and description information for a Bib graph and then create a new graph with this information as well as the original data from the Bib graph.

Datasets:
* WorldCat Bib: graph to host on GraphDB at https://graphdb.dumontierlab.com
* Wikidata: hosted on Virtuoso at https://query.wikidata.org/sparql


```SPARQL
PREFIX schema: <http://schema.org/> 
PREFIX library: <http://purl.org/library/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> 
PREFIX wdt: <http://www.wikidata.org/prop/direct/> 
 
CONSTRUCT
{
<http://www.worldcat.org/oclc/1210> ?p ?o.
<http://www.worldcat.org/oclc/1210> schema:name ?name.
<http://www.worldcat.org/oclc/1210> schema:description ?description .
}

FROM <http://www.worldcat.org/oclc/1210> 
WHERE {  
<http://www.worldcat.org/oclc/1210> ?p ?o
<http://www.worldcat.org/oclc/1210> library:oclcnum ?oclcNumber.  
 
SERVICE <https://query.wikidata.org/sparql> {  
    ?work wdt:P243 ?oclcNumber. 
    ?work rdfs:label ?name.
    OPTIONAL {
        ?work schema:description ?description .
    }
} 
}```

