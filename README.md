# did-summarizer
Linked Data summarizer driven by Decentralized Identifiers (DIDs)

Funded by CLARIAH project

The main aim of the Summarizer service is to gain an overview about which vocabularies are already used in CLARIAH, or might be useful in CLARIAH. The core is to build an analyzing pipeline containing data collection, vocabulary analysis, report. While concentrating on the automatised pipeline, we also indicate at which point where expert/manual curation is needed. 

To bridge between different knowledge domains it is needed to find communalities, cross-walks, mappings between vocabularies. A prerequisite for this is to gain an overview what vocabularies exist (VOCABULARY part) and how they are used (DATASET part). Despite of many existing registries this problem is by no means solved, nor are there standard, off-the shelf solutions for gaining such an overview.

Decentralized identifiers (DIDs) are being used to create resolvable globally accessible unique and persistent identifiers to support various Linked Data tasks in FAIR way:
* assign DID to SPARQL query to make it persistent 
* cache vocabulary concept content and relationships
* store and get statistics of usage for vocabulary concepts available in the time dimension, just like the Internet Archive
* assign unique DID to other services used in CLARIAH pipelines
...

# Installation

```
cp .env_sample .env
docker network create traefik
export traefikhost=somedomain.dans.knaw.nl
docker-compose up -d
cd ./distros/DID
docker-compose up -d
```

# Testing
Use simple test to send strings and generate DIDs for every uri sample:
```
bash ./distros/DID/tests/simple-test.sh 
```
Response should provide the list of URIs with corresponding DIDs:
```
{"uri_1":"did:oyd:zQmXYw1zsGNREhp9aWVdPxqozCYwRaCvoh7nRYwSsWqQGJK","uri_2":"did:oyd:zQmZRG1MrrkKMTSPiuxv7C1oWix7bWufpwg6NntXK7DgQmQ","uri_3":"did:oyd:zQmU1B3Pf1nBgKVCSqQgLSKrkb7fKHzXTzEvdf4vuC5VroD"}
```
Example of request to resolve some DID:
```
curl https://dev.uniresolver.io/1.0/identifiers/did:oyd:zQmXYw1zsGNREhp9aWVdPxqozCYwRaCvoh7nRYwSsWqQGJK
``` 

# Summarizer example

```
curl -X 'GET' \
  'http://0.0.0.0:8001/summarizer?url=https%3A%2F%2Fraw.githubusercontent.com%2FAKSW%2Fdssn.rdf%2Fmaster%2Fnamespace.ttl' \
  -H 'accept: application/json'
```

Response body:
```
{
  "statements": {
    "statements": 119,
    "unique objects": 64,
    "unique predicates": 25,
    "unique subjects": 22
  },
  "prefixes": {
    "http://www.w3.org/2000/01/rdf-schema#": "rdfs",
    "http://www.w3.org/2002/07/owl#": "owl",
    "http://www.w3.org/2004/02/skos/core#": "skos",
    "http://xmlns.com/foaf/0.1/": "foaf",
    "http://rdfs.org/sioc/ns#": "sioc",
    "http://usefulinc.com/ns/doap#": "doap",
    "http://www.w3.org/2003/06/sw-vocab-status/ns#": "vs",
    "http://purl.org/dc/terms/": "dct",
    "http://purl.org/net/dssn/": "dssn"
  },
  "stats": {
    "dssn": 126,
    "rdfs": 60,
    "vs": 13,
    "doap": 1,
    "foaf": 20,
    "owl": 15,
    "dct": 1,
    "sioc": 11,
    "skos": 1
  }
}
```

# Vocabulary recommender integration

Vocabulary Recommender Command-line interface (CLI) was developed by [Triply](http://triply.cc) and provides a recommendation interface which returns relevant Internationalized Resource Identifiers (IRIs) based on the search input. It works with SPARQL or Elasticsearch endpoints which contain relevant vocabulary datasets. [DANS](https://dans.knaw.nl/en) has created a service out of it.

Usage example:

```
curl -X 'GET' \
  'http://0.0.0.0:8001/recommend?searchTerm=person&searchClass=class' \
  -H 'accept: application/json'
```

Response:

```
[
  [
    {
      "searchTerm": "person",
      "vocabs": [
        "https://w3id.org/pnv#"
      ],
      "homogeneous": [
        {
          "iri": "https://w3id.org/pnv#Person",
          "score": 1,
          "vocabPrefix": "https://w3id.org/pnv#",
          "vocabDomain": "https://w3id.org/pnv#",
          "description": "A Person is a human being whose individual existence can somehow be documented",
          "category": "class"
        }
      ],
      "single": [
        {
          "iri": "http://xmlns.com/foaf/0.1/Person",
          "score": 0.8853529652138054,
          "vocabPrefix": "foaf",
          "vocabDomain": "http://xmlns.com/foaf/0.1/",
          "description": "A person.",
          "category": "class"
        },
        {
          "iri": "https://schema.org/Person",
          "score": 0.7432885949870411,
          "vocabPrefix": "https://schema.org/",
          "vocabDomain": "https://schema.org/",
          "description": "A person (alive, dead, undead, or fictional).",
          "category": "class"
        },
...
]
```

You can find more information about supported parameters and use cases in CLARIAH Vocabulary Recommender [Github repo](https://github.com/CLARIAH/vocabulary-recommender).

# DID names

By default DID containers available for different ontologies:
* persons -> https://persons.did.zandbak.dans.knaw.nl
* locations -> https://locations.did.zandbak.dans.knaw.nl
* organizations -> https://organizations.did.zandbak.dans.knaw.nl
* uri -> https://uri.did.zandbak.dans.knaw.nl

API endpoint with cache and resolver:
* api -> https://api.did.zandbak.dans.knaw.nl/docs

# Cache and storage

All concepts are being cached in RAM using Redis framework and preserved in MongoDB database. After every restart the key:value pair for URI:DID reindexed and available for lookup in the cache. It should be possible to move all DIDs data from one network to another without too much efforts.  

# Use cases

The idea of DID summarizer is to assign unique globally resolvable decentralized identifier DID to any string, particulary to URLs and URIs. 
Possible usage:
* Storing all inbound and outbound links for the specific web page defined by URL
* Assign DIDs for any content and archive it in the decentralized network with access by private key 
* Build sustainable knowledge graph compliant with FAIR principles: every edge and vertice should have their own unique DID identifier 
