# tree-diffs
Experiments showing (taxonomic) tree diffs.

The GBIF Checklist Bank and backbone tools can dump datasets in simple hierarchical text files or create an xml representation. 

## Tree text files
Tree text files simply list all names in a taxonomic order, sorting all children and synonyms by their rank and then name string.
In addition to the name itself the tree provides a few other properties per name:
 - the rank is given in brackets after the name
 - synonym names are prefixed with a ***
 - basionyms are prefixed with a *$*

```
Mantodea [order]
  Hymenopodidae [family]
    Amphecostephanus [genus]
      Amphecostephanus rex [species]
    Galinthias [genus]
      Galinthias amoena [species]
        *Galinthias hyalina [species]
      $Galinthias meruensis [species]
        *Galinthias usambarica [species]
        *Oxypilus meruensis [species]
        *Oxypilus nigericus [species]
      Galinthias occidentalis [species]
```

The text files can either use simple canonical names (default) or names with the full authorship indicated by filenames containing **-full**.

### PostgreSQL generated trees
With recursive sql statements one can generate an indented txt output just like above. For the Catalog of Life it looks like this down to the rank of order:

```
WITH RECURSIVE taxa_rec AS (
  (SELECT 1 AS depth, ARRAY[n.canonical_name] AS path, u.id, n.scientific_name || ' [' || lower(u.rank::text) || ']' AS name
   FROM   name_usage u JOIN name n ON u.name_fk=n.id
   WHERE  u.dataset_key='7ddf754f-d193-4cc9-b351-99906754a03b' AND u.parent_fk IS NULL
  )    
   UNION ALL
   SELECT r.depth + 1, r.path || n.canonical_name, u.id, repeat('  ',r.depth) || n.scientific_name || ' [' || lower(u.rank::text) || ']'
   FROM   taxa_rec r 
   JOIN   name_usage u ON u.parent_fk = r.id JOIN name n ON u.name_fk=n.id
   WHERE  u.rank <= 'ORDER'
)
SELECT name
FROM   taxa_rec
ORDER  BY path;
```

To export the results as txt files it is easiest to create a view and then copy that to a file.



## Tree xml files
tbd
