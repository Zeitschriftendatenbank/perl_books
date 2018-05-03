# Exercise 1 - Set up a LDF server

Download project files:

```terminal
$ cd /home/catmandu/
$ wget http://jorol.de/2018-ELAG.zip
$ unzip 2018-ELAG.zip
```

In the terminal change the directory to:

```terminal
$ cd 2018-ELAG
```

Create a new HDT file for `perl_authors.ttl` with the command:

```terminal
$ rdf2hdt -f turtle perl_authors.ttl perl_authors.hdt
```

Edit config file:

```terminal
$ nano config.json
```

Content of `config.json`:

```json
{
  "title": "ELAG18 Linked Data Fragments server",
  "datasources": {
    "camel": {
      "title": "The camel books",
      "description": "Just some Perl books",
      "type": "HdtDatasource",
      "settings": {
        "file": "../LinkedDataFragments/data/camel.hdt"
      }
    },
    "perl_authors": {
      "title": "Perl authors",
      "description": "Authors of Perl books",
      "type": "HdtDatasource",
      "settings": {
        "file": "perl_authors.hdt"
      }
    }
  },
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

Start the LDF server with the command:

```terminal
$ ldf-server config.json 3000
```

Open in the Virtual Box a web browser and surf to the location <http://localhost:3000/perl_authors>.

Search the `perl_authors` dataset for the object `"Wall, Larry"` and get more information about the subject.

Test if you can access the LDF server via the command line. In the terminal open an new window via the menu `System Tools >> LXTerminal`.

Execute the following command to extract all RDF triples from the new HDT store:

```terminal
$ catmandu convert RDF --url http://127.0.0.1:3000/perl_authors --sparql 'select * where { ?s ?p ?o}'
```

Create a file named `authors.sparql` with the following content:

```sparql
select * where {    
    ?record <http://d-nb.info/standards/elementset/gnd#preferredNameForThePerson> "Wall, Larry" 
}  
```

Run this SPARQL query with the following command:

```terminal
$ catmandu convert RDF --url http://127.0.0.1:3000/perl_authors --sparql authors.sparql
```

Keep that terminal open and the server running for the following exercises.
