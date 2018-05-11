# Exercise 3 - Convert MARC to RDF

Use the RDF exporter with the option `--type Turtle` to convert the output:

```terminal
$ catmandu convert MARC to RDF --fix marc2dc.fix --type Turtle < perl_books.mrc
```

Create a Fix `author_lookup.fix` to lookup `dc_contributors` with our `perl_authors` LDF server datasource from Exercise 1. 
We first iterate over the dc_contributor list and then lookup strings like  "Wall, Larry" using the rdf_ldf_statements() fix method.

Hint: When typing spare the comments (# ...)

```perl
# only if contributor exists
if exists(dc_contributor)
    
    # iterate over all contributor values
    # varibale 'current' is the current contributor
    do list(path:dc_contributor, var:current)
    
        # a lookup variable saves the current name
        copy_field(current,lookup)

        # Double quotes needed for request in LDF server, e.g. 'Wall, Larry' --> '"Wall, Larry"'
        prepend(lookup,'"')
        append(lookup,'"')

        # let Catmandu make the Triple Pattern Fragments request
        # on success variable lookup will be overwritten
        rdf_ldf_statements(lookup, url:"http://127.0.0.1:3000/perl_authors", predicate:"http://d-nb.info/standards/elementset/gnd#preferredNameForThePerson")
        
        # on success lookup should be a list
        if exists(lookup.0)
            
            # copy first list element to the current list element
            # pretty sure the first list element is the right one ;-)
            copy_field(lookup.0,dct_contributor.$append)
        end
    end
end

# we don't need lookup anymore
remove_field(lookup)
```

Make sure that the LDF server with the `perl_authors` datasource from Exercise 1 is still running.

Rerun the RDF conversion with the author lookup and store the result in a file:

```terminal
$ catmandu convert MARC to RDF --fix marc2dc.fix --fix author_lookup.fix --type Turtle < perl_books.mrc > perl_books.ttl
```

Check if the lookup was successful (enriched by URIs of the German Authority File GND):

```terminal
$  grep 'http://d-nb.info/gnd/138937079' perl_books.ttl
```

Create a new HDT file for `perl_books.ttl` with the command:

```terminal
$ rdf2hdt -f turtle perl_books.ttl perl_books.hdt
```

Edit config:

```terminal
$ nano config.json
```

```json
{
  "title": "SWIB16 Linked Data Fragments server",
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
    },
    "perl_books": {
      "title": "Perl books",
      "description": "Perl books",
      "type": "HdtDatasource",
      "settings": {
        "file": "perl_books.hdt"
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

Go to the first terminal and stop the LDF server (`CTRL + C`) and start again:

```terminal
$ ldf-server config.json 3000
```

Open in the Virtual Box a web browser and surf to the location <http://127.0.0.1:3000/perl_books>.

Search for the number of records in the books collection which have `<http://d-nb.info/gnd/138937079>` as object.

Test if you can access the LDF server via the command line.  In the Terminal open an new terminal via the menu `System Tools >> LXTerminal`.

Execute the following command to extract all RDF triples from the new HDT store:

```terminal
$ catmandu convert RDF --url http://127.0.0.1:3000/perl_books --sparql 'select * where { ?s ?p ?o}'
```

Create a file named `books.sparql` with the following content:

```sparql
select * where {
    ?record <http://purl.org/dc/elements/1.1/contributor> <http://d-nb.info/gnd/138937079>
} 
```

Run this SPARQL query with the following command:

```terminal
$ catmandu convert RDF --url http://127.0.0.1:3000/perl_books --sparql books.sparql
```
