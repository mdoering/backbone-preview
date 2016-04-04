# GBIF Backbone Identifier
Some queries comparing the old 2013 backbone with the latest build from 2016-03-11. It is important that we keep identifiers stable, see:

 - http://dev.gbif.org/issues/browse/POR-2786
 - http://dev.gbif.org/issues/browse/POR-3060

## SETUP
```
create table prod_usage (id int primary key, rank rank, kingdom int, status text, deleted timestamp, name text, canonical text, new boolean);
create index on prod_usage (name);
create index on prod_usage (canonical);
create table uat_usage (like prod_usage INCLUDING INDEXES);
\copy (select u.id, u.rank, u.kingdom_fk, u.status, u.deleted, scientific_name, canonical_name, false) from name_usage u join name n on name_fk=n.id where dataset_key='d7dddbf4-2cf0-4f39-9b2a-bb099caae36c') to 'uat.txt'
\copy (select u.id, u.rank, u.kingdom_fk, u.status, u.deleted, scientific_name, canonical_name, false) from name_usage u join name n on name_fk=n.id where dataset_key='d7dddbf4-2cf0-4f39-9b2a-bb099caae36c') to 'prod.txt'
\copy uat_usage from 'uat.txt'
\copy prod_usage from 'prod.txt'
update uat_usage set new = true;
update uat_usage u set new = false FROM prod_usage AS p WHERE p.id = u.id;
```

# ID Metrics
## TOTAL

```
select count(*) from uat_usage;
select count(*) from prod_usage;

uat :  5.354.346
prod: 4.416.347
```

## ACCEPTED

```
select count(*) from uat_usage where status='ACCEPTED';
select count(*) from prod_usage where status='ACCEPTED';

uat : 2.816.335
prod: 3.164.998
```

## SHARED KEYS

```
select count(*) from uat_usage u join prod_usage p on u.id=p.id where u.deleted is null;

shared & not deleted: 4.112.811
```
## UAT KEYS

```
select count(*) from uat_usage where deleted is not null;
select count(*) from uat_usage where deleted is null;
select count(*) from uat_usage where new;

deleted :   303.536
current : 5.050.810
new     :   937.999
```


# SUSPICOUS TAXA
check for matching names but different ids on UAT vs PROD

### link on canonical name
```
select u.id, u.rank, u.name, p.id, p.rank, p.name FROM uat_usage u join prod_usage p ON u.canonical=p.canonical WHERE u.new and u.kingdom=p.kingdom order by u.name limit 20;
```

```
   id    |  rank   |                       name                       |   id    |  rank   |                  name                   
---------+---------+--------------------------------------------------+---------+---------+-----------------------------------------
 7673341 | SPECIES | "Mycena pseudocrispula """"f. bisporique"""""    | 5240717 | SPECIES | Mycena pseudocrispula Kühner, 1938
 7350216 | SPECIES | "Urnula craterium (Schw.                         | 5258535 | SPECIES | Urnula craterium (Schwein.) Fr., 1851
 8097329 | GENUS   | +Crataegomespilus Simon-Louis ex Bellair, 1899   | 3026356 | GENUS   | Crataegomespilus Simon-Louis ex Bellair
 7381534 | GENUS   | +Pyrocydonia H.K.A. Winkler ex L.L. Daniel, 1913 | 3000718 | GENUS   | Pyrocydonia Rehder
 7884109 | GENUS   | Aa Rchb. f.                                      | 2850872 | GENUS   | Aa H.G. Reichenbach, 1854
 7767226 | SPECIES | Aa calceata Rchb.f.                              | 5325297 | SPECIES | Aa calceata (Rchb.f.) Schltr.
 8218017 | SPECIES | Aa erosa Rchb.f.                                 | 5325294 | SPECIES | Aa erosa (Rchb.f.) Schltr.
 7920518 | SPECIES | Aa filamentosa M.L.Ortiz                         | 5308802 | SPECIES | Aa filamentosa Mansf.
 7541683 | SPECIES | Aa gymnandra Rchb.f.                             | 5325279 | SPECIES | Aa gymnandra (Rchb.f.) Schltr.
 7769242 | SPECIES | Aa hieronymi Cogn.                               | 5325292 | SPECIES | Aa hieronymi (Cogn.) Schltr.
 8215893 | SPECIES | Aa inaequalis Rchb.f.                            | 5325273 | SPECIES | Aa inaequalis (Rchb.f.) Schltr.
 7832902 | SPECIES | Aa leucantha Rchb.f.                             | 5325274 | SPECIES | Aa leucantha (Rchb.f.) Schltr.
 8260536 | SPECIES | Aa mandonii Rchb.f.                              | 5325290 | SPECIES | Aa mandonii (Rchb.f.) Schltr.
 7458722 | SPECIES | Aa matthewsii Rchb.f.                            | 5325271 | SPECIES | Aa matthewsii (Rchb.f.) Schltr.
 7930687 | SPECIES | Aa nervosa Kraenzl.                              | 5325280 | SPECIES | Aa nervosa (Kraenzl.) Schltr.
 7391285 | SPECIES | Aa rostrata (Rchb.f.) J.B.Neves                  | 5308800 | SPECIES | Aa rostrata (Rchb.f.) Schltr.
 7882934 | SPECIES | Aa weddeliana Rchb.f.                            | 5543254 | SPECIES | Aa weddeliana Schltr.
 7973631 | GENUS   | Aalius Rumph.                                    | 3076834 | GENUS   | Aalius Rumph. ex Lam.
 8024259 | GENUS   | Aalius Rumphius ex O. Kuntze, 1891               | 3076834 | GENUS   | Aalius Rumph. ex Lam.
 7956398 | GENUS   | Aancistroger Bey-Bienko, 1957                    | 1725827 | GENUS   | Aancistroger Bei-Bienko, 1957
```

This looks mostly good. The names are indeed different, the current author is different. Either because it was the basionym or it was an ex author which we only recently handle correctly.

Problematic:

```
 7956398 | GENUS      | Aancistroger Bey-Bienko, 1957                                                | 1725827 | GENUS      | Aancistroger Bei-Bienko, 1957
 8006139 | GENUS      | Aaptolasma Newman & Ross, 1971                                               | 2115823 | GENUS      | Aaptolasma Newman & Ross, 1971
 8128511 | SPECIES    | Abacetus (Astigis) aeneus (Dejean, 1828)                                     | 4473291 | SPECIES    | Abacetus (Astigis) aeneus (Dejean, 1828)
 7666198 | VARIETY    | Abies fargesii Franch. var. faxoniana (Rehder & Wilis.) Liu                  | 2685640 | VARIETY    | Abies fargesii var. faxoniana (Rehd. & E.H. Wilson) T. S. Liu
 7666198 | VARIETY    | Abies fargesii Franch. var. faxoniana (Rehder & Wilis.) Liu                  | 6408970 | SUBSPECIES | Abies fargesii subsp. faxoniana (Rehder & E.H.Wilson) Silba
 
```
 

### link on scientific name
These names appear to have a new id assigned mostly wrongly. They need more investigation:

```
select u.id, u.rank, u.name, p.id, p.rank, p.name FROM uat_usage u join prod_usage p ON u.name=p.name WHERE u.new and u.kingdom=p.kingdom order by u.name limit 1000;
```

```
8006139 | GENUS      | Aaptolasma Newman & Ross, 1971                                    | 2115823 | GENUS      | Aaptolasma Newman & Ross, 1971
 7880207 | SPECIES    | Abaca bunchy top virus                                            | 6875715 | SPECIES    | Abaca bunchy top virus
 8128511 | SPECIES    | Abacetus (Astigis) aeneus (Dejean, 1828)                          | 4473291 | SPECIES    | Abacetus (Astigis) aeneus (Dejean, 1828)
 7917612 | GENUS      | Abantiades                                                        | 1228122 | GENUS      | Abantiades
 8177957 | GENUS      | Abaxitrella Gorochov, 2002                                        | 1719013 | GENUS      | Abaxitrella Gorochov, 2002
 7875529 | GENUS      | Abderites Ameghino, 1887                                          | 4826382 | GENUS      | Abderites Ameghino, 1887
 7953223 | GENUS      | Abebaeus                                                          | 1192988 | GENUS      | Abebaeus
 7964788 | GENUS      | Abergasilis                                                       | 2110454 | GENUS      | Abergasilis
 7936260 | GENUS      | Aberratylus Bousfield & Kendall, 1994                             | 2216944 | GENUS      | Aberratylus Bousfield & Kendall, 1994
 8244697 | SPECIES    | Abgrallaspis colorata (Cockerell, 1893)                           | 2088159 | SPECIES    | Abgrallaspis colorata (Cockerell, 1893)
 7691007 | SPECIES    | Abgrallaspis flabellata (Ferris, 1938)                            | 2088164 | SPECIES    | Abgrallaspis flabellata (Ferris, 1938)
 8029205 | SPECIES    | Abgrallaspis furcillae (Brain, 1918)                              | 2088199 | SPECIES    | Abgrallaspis furcillae (Brain, 1918)
 8120282 | SPECIES    | Abgrallaspis ithacae (Ferris, 1938)                               | 2088170 | SPECIES    | Abgrallaspis ithacae (Ferris, 1938)
 8095333 | SPECIES    | Abgrallaspis ruebsaameni (Cockerell, 1902)                        | 2088203 | SPECIES    | Abgrallaspis ruebsaameni (Cockerell, 1902)
 7515274 | SPECIES    | Abida pyrenaearia (Michaud, 1831)                                 | 4567005 | SPECIES    | Abida pyrenaearia (Michaud, 1831)
 8283469 | SPECIES    | Abies kaempferi (Lamb.) Lindl.                                    | 2686163 | SPECIES    | Abies kaempferi (Lamb.) Lindl.
 8229521 | SPECIES    | Abies lindleyana Roezl ex Carrière                                | 2685838 | SPECIES    | Abies lindleyana Roezl ex Carrière
 7669260 | GENUS      | Abisares Stål, 1878                                               | 1708119 | GENUS      | Abisares Stål, 1878
 8281423 | SPECIES    | Ablabes melanocephalus Günther, 1858                              | 2456255 | SPECIES    | Ablabes melanocephalus Günther, 1858
 8281918 | SPECIES    | Ablepharus boutoni Boulenger, 1887                                | 2463588 | SPECIES    | Ablepharus boutoni Boulenger, 1887
 8281962 | SPECIES    | Ablepharus festae Peracca, 1894                                   | 2461278 | SPECIES    | Ablepharus festae Peracca, 1894
 7607602 | SPECIES    | Ablepharus rueppellii Schmidtler, 1997                            | 2461304 | SPECIES    | Ablepharus rueppellii Schmidtler, 1997
 8008620 | GENUS      | Abracris Walker, 1870                                             | 1703594 | GENUS      | Abracris Walker, 1870
 8282180 | GENUS      | Abramidopsis Siebold, 1863                                        | 4837595 | GENUS      | Abramidopsis Siebold, 1863
 8095457 | GENUS      | Absolonia Roewer, 1915                                            | 3253424 | GENUS      | Absolonia Roewer, 1915
 8084766 | SPECIES    | Abutilon mosaic virus                                             | 6877059 | SPECIES    | Abutilon mosaic virus
 8283488 | SUBSPECIES | Abutilon sylvaticum subsp. buchtienii R.E.Fr.                     | 3935714 | SUBSPECIES | Abutilon sylvaticum subsp. buchtienii R.E.Fr.
 7559503 | SPECIES    | Abutilon yellows virus                                            | 6875846 | SPECIES    | Abutilon yellows virus
 8050659 | GENUS      | Acacallis                                                         | 1178554 | GENUS      | Acacallis
 8285178 | SPECIES    | Acacia angico Mart.                                               | 2941630 | SPECIES    | Acacia angico Mart.
 8259770 | VARIETY    | Acacia angustissima var. angustissima                             | 7225100 | VARIETY    | Acacia angustissima var. angustissima
 8285097 | SPECIES    | Acacia campbellii Arn.                                            | 2979132 | SPECIES    | Acacia campbellii Arn.
 8014154 | VARIETY    | Acacia decurrens var. decurrens                                   | 7225024 | VARIETY    | Acacia decurrens var. decurrens
 8285153 | SPECIES    | Acacia elliptica Benth.                                           | 2979101 | SPECIES    | Acacia elliptica Benth.
 8285192 | SPECIES    | Acacia hirsuta Schltdl.                                           | 2979198 | SPECIES    | Acacia hirsuta Schltdl.
 8285137 | SPECIES    | Acacia pilosa DC., p.p.                                           | 2978702 | SPECIES    | Acacia pilosa DC., p.p.
 8058935 | SPECIES    | Acacia pilosa DC., p.p.                                           | 2978702 | SPECIES    | Acacia pilosa DC., p.p.
 8285100 | SPECIES    | Acacia pseudowightii Thoth.                                       | 2979135 | SPECIES    | Acacia pseudowightii Thoth.
 8285155 | SPECIES    | Acacia spathulata Benth., p.p.                                    | 2980267 | SPECIES    | Acacia spathulata Benth., p.p.
 8285325 | SPECIES    | Acacia usumacintensis Lundell                                     | 2965821 | SPECIES    | Acacia usumacintensis Lundell
 7424294 | SPECIES    | Acacia verticillata (L'Her.) Willd.                               | 2979292 | SPECIES    | Acacia verticillata (L'Her.) Willd.
 8285324 | SPECIES    | Acacia vogeliana Steud.                                           | 2965843 | SPECIES    | Acacia vogeliana Steud.
```
