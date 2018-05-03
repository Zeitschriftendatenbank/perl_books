# Exercise 2 - Processing MARC with Catmandu

Change directory and show the content of the project directory:

```terminal
$ cd /home/catmandu/2018-ELAG
$ ls -l
```

Inspect `perl_books.mrc` in two different ways:

... with `less`

```terminal
$ less perl_books.mrc
```

Use the `space` key to browse through a long file and the `q` key to exit the `less` command.

... with `catmandu`:

```terminal
$ catmandu convert MARC to JSON < perl_books.mrc
$ catmandu convert MARC to YAML < perl_books.mrc
```

Map subfield `a` from field `245` to `dc_title`:

```terminal
$ catmandu convert MARC to YAML  --fix 'marc_map(245a,title);retain(title)' < perl_books.mrc
```

Create a Fix script `marc_title.fix` to extract the title fields from the a MARC input and delete all the rest:

```perl
marc_map(245,title)
retain(title)
```

Execute this Fix script on a MARC input and generate CSV

```terminal
$ catmandu convert MARC to YAML --fix marc_title.fix < perl_books.mrc
```

Create a new Fix script `marc2dc.fix` and execute it again.

Hint: While typing spare the comments (# ...)

```perl
# DC - http://www.loc.gov/marc/marc2dc.html

prepend(_id,'http://my.perl.books/record/')

# Identifier
marc_map(0209,dc_identifier,split:1)
# Transform identifier to URIs
if exists(dc_identifier)    
    do maybe()     
        isbn13(dc_identifier.*)
    end    
    prepend(dc_identifier.*,'urn:isbn:')
    sort_field(dc_identifier,uniq:1)
end

# DDC subjects
marc_map(082a,dc_subject,split:1)

# Contributor
marc_map(100a,dc_contributor,split:1)
marc_map(700a,dc_contributor,split:1)
marc_map(720a,dc_contributor,split:1)
flatten(dc_contributor)

# Title
marc_map(245a,dc_title)

# Publisher
marc_map(260ab,dc_publisher.$append,join:' : ')

# Language
marc_map(008/35-37,dc_language,split:1)
marc_map(041abdefghj,dc_language,split:1)
flatten(dc_language)
sort_field(dc_language,uniq:1) 

# Dates
marc_map(260c,dc_date,split:1)
# Clean dates and get unique dates
replace_all(dc_date.*,'[^0-9]+','') 
sort_field(dc_date,uniq:1)

remove_field(record)
```

```terminal
$ catmandu convert MARC to YAML --fix marc2dc.fix < perl_books.mrc
```