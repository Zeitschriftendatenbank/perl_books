# Exercise 4 - Federated query with Linked Data Framents Client

Change the directory to:

```terminal
$ cd /home/catmandu/LinkedDataFragments/client
```

Edit settings:

```terminal
$ nano settings.json
```

Add your `perl_books.hdt` Linked Data Framents source to the clients `settings.json`:

```json
{
  "datasources": [
   {
      "name": "Perl books",
      "url": "http://127.0.0.1:3000/perl_books"
    },
   {
      "name": "Perl authors",
      "url": "http://127.0.0.1:3000/perl_authors"
    },
    {
      "name": "Virtual International Authority File (VIAF)",
      "url": "http://data.linkeddatafragments.org/viaf"
    }
  ],
  "prefixes": {
    "rdf":         "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "rdfs":        "http://www.w3.org/2000/01/rdf-schema#",
    "owl":         "http://www.w3.org/2002/07/owl#",
    "skos":        "http://www.w3.org/2004/02/skos/core#",
    "xsd":         "http://www.w3.org/2001/XMLSchema#",
    "dc":          "http://purl.org/dc/elements/1.1/",
    "dct":         "http://purl.org/dc/terms/",
    "foaf":        "http://xmlns.com/foaf/0.1/",
    "dbpedia":     "http://dbpedia.org/resource/",
    "dbpedia-owl": "http://dbpedia.org/ontology/",
    "dbpprop":     "http://dbpedia.org/property/",
    "schema":      "http://schema.org/",
    "gndo":        "http://d-nb.info/standards/elementset/gnd#",
    "gnd":         "http://d-nb.info/gnd/",
    "viaf":        "http://viaf.org/viaf/"
  }
}

```

Make sure that the LDF server with the `perl_authors` and `perl_books` datasources from Exercise 3 is still running.

Start a new Linked Data Fragments client with your settings:

```terminal
$ npm start
```

Open in the Virtual Box a web browser and surf to the location <http://127.0.0.1:3001>. In the dropdown list with the label `Choose datasources` the browser version of the Linked Data Framents client should now contain your recently added datasource `Perl Books`.

Choose `Perl Books` only as datasouce. And make a SPARQL request for remaining contributor names:

```
SELECT * WHERE {
    ?book dc:contributor ?name .
}
```

Add 'Virtual International Authority File (VIAF)' to the datasouces and query for VIAF-URIs with your first federated Linked Data Framents query:

```
SELECT * WHERE {
      ?book dc:contributor ?name .

      # this connects contributor names from perl_books with VIAF IDs
      ?viaf schema:name ?name .
}
```

Expand this query with URIs from VIAFs sameAs links:

```
SELECT * WHERE {
    
    ?book dc:contributor ?name .

    ?viaf schema:name ?name .
  
    # get the sameAs links
    ?viaf schema:sameAs ?uri .
}
```

Alter the query to enrich your datasource with sameAs URIs from VIAF

```
CONSTRUCT {
    ?book dct:contributor ?viaf .
    ?book dct:contributor ?uri .
    ?book ?p ?o .
}
WHERE {
    ?book dct:contributor ?gnd .
    ?book ?p ?o .
    ?viaf schema:name ?gnd .
    ?viaf schema:sameAs ?uri .
}
```

Copy this query, create a file `enrich.sparl`, paste in the query and set a `dct` PREFIX:

```
PREFIX dct: <http://purl.org/dc/terms/>

CONSTRUCT {
    ?book dct:contributor ?viaf .
    ?book dct:contributor ?uri .
    ?book ?p ?o .
}
WHERE {
    ?book dct:contributor ?gnd .
    ?book ?p ?o .
    ?viaf schema:name ?gnd .
    ?viaf schema:sameAs ?uri .
}
```

On the command line go to directory where file `enrich.sparql` resides.

Enrich your data with the Linked Data Fragements command line client:

```terminal
$ ldf-client http://data.linkeddatafragments.org/viaf http://127.0.0.1:3000/perl_books -f enrich.sparql > perl_books_enriched.ttl
``` 