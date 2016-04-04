# Updating the GBIF Backbone
The taxonomy employed by GBIF for organising all occurrences into a consistent view has been unchanged for a while and dates back to 2013. We have been working on a replacement for some time and like to introduce a preview in this post. The work in rather complex and tries to establish an automated process of building a new backbone which we aim to run on a regular, probably quaterly basis. We like to release the new taxonomy rather soon and improve the backbone iteratively. Large regressions should be avoided initially, but it is quite hard to evaluate all the changes from 2 larges taxonomies with 4 - 5 million names each. We therefore seek feedback and help in discovering oddities of the new backbone.

## Relevance & Challenges
Every occurrence record in GBIF is matched to a taxon in the backbone. Because occurrence records in GBIF cover the entire Life and names come from all possible and often outdated taxonomies it is  important to have the broadest coverage of names as possible. We also deal with fossil names, extinct taxa and due to advanced digital publishing even names that have just been described a week before the data is indexed at GBIF.

The Taxonomic Backbone provides a single classification and a synonymy that we use to inform our systems when creating maps, providing metrics or even when you do a plain occurrence search. It is also used to crosslink names from checklist datasets to each other.

## The Origins
The very first taxonomy that GBIF used was based on the Catalog of Life. As it did not even cover half the names we found in GBIF occurrences, the cleaned occurrence names were merged into the nub. As the nub grew we never deleted names and increasingly faced more and more redundant names with slightly different classifications. It was time for a different procedure.

## The Current Backbone
The current version of the backbone was built in July 2013. It is largely based on the Catalog of Life from 2012 and has folded in names from [39 further taxonomic sources](nub-live/sources.md). It was build with an automated process that made use of selected checklists from the GBIF ChecklistBank in a prioritised order. The Catalog of Life was still the starting point and provided the higher classification down to orders. The [Interim Register of Marine and Nonmarine Genera](http://www.gbif-uat.org/dataset/714c64e3-2dc1-4bb7-91e4-54be5af4da12) was used as the single reference list for homonym genera. Otherwise only a single version of any name was allowed to exist in the backbone, even if they differ in authorship. 

### Current issues
We kept track of [nearly 150 reported issues](http://dev.gbif.org/issues/issues/?jql=labels%20%3D%20nub). Some of the main issues showing up regularly that we wanted to address were:

 - Enable an [automated build process](http://dev.gbif.org/issues/browse/POR-2467) so we can use the latest Catalog of Life and other sources to capture newly described or currently missing names
 - It was impossible to have [synonyms using the same canonical name but with different authors](http://dev.gbif.org/issues/browse/POR-353). This means [*Poa pubescens*](http://www.gbif.org/species/4113236) was always considered a synonym of *Poa pratensis* L. when in fact *Poa pubescens* R.Br. is considered a synonym of Eragrostis pubescens (R.Br.) Steud. - Some families contain far too many accepted species and hardly any synonyms. Especially for plants the Catalog of Life was surprisingly sparsely populated and we heavily relied on IPNI names. For example the family [*Cactaceae* has 12.062 accepted species](http://dev.gbif.org/issues/browse/POR-1389) in GBIF while The Plant List recognizes just 2.233. - Many accepted names are based on the same basionym, for example the current backbone considers both [*Sulcorebutia breviflora* Backeb.](http://www.gbif.org/species/7283318) and [*Weingartia breviflora* (Backeb.) Hentzschel & K.Augustin](http://www.gbif.org/species/7281391) as accepted taxa. - Relying purely on IRMNG for homonyms meant that homonyms which IRMNG was not aware of to be conflated. On the other hand there are many genera in IRMNG - and thus in the backbone - that are hardly used anywhere, creating confusion and many empty genera without any species in our backbone.

## The New Backbone
The new backbone is available for [preview in our test environment](http://www.gbif-uat.org/dataset/d7dddbf4-2cf0-4f39-9b2a-bb099caae36c). In order to review the new backbone and compare it to the [previous version](http://www.gbif.org/dataset/d7dddbf4-2cf0-4f39-9b2a-bb099caae36c) we provide a few tools with a different focus:

 - **Stable ID report**: We have joined the old and new backbone names to each other and [compared their identifiers](nub/stable-ids.md). When joining on the full scientific name there is still an issue with changing identifiers which we do not know the cause yet.
 - **Tree Diffs**: For comparing the higher classification we used a [tool from Rod Page](http://iphylo.blogspot.dk/2015/12/visualising-difference-between-two.html) to [diff the tree down to families](nub/families.html?raw=true). There are surprisingly many changes, but all of them stem from an evolving Catalog of Life or the changed Algae classification.
 - **Nub Browser**: For comparing actual species and also reviewing the impact of the changed taxonomy on the GBIF occurrences we developed a [new Backbone Browser](http://mdoering.github.io/nub-browser/app/#/) sitting on top of our existing API. Our test environment has a complete copy of the current GBIF occurrence index which we have reprocessed to use the new backbone. This also includes all maps and [metrics](http://mdoering.github.io/nub-browser/app/#/metrics) which we show in the new browser.

### Sources
We manually curate a [list of priority ordered checklist datasets](https://github.com/gbif/checklistbank/blob/master/checklistbank-nub/nub-sources.tsv) that we use to build the taxonomy. Three datasets are treated slightly special:

 1. [GBIF Backbone Patch](http://www.gbif-uat.org/dataset/daacce49-b206-469b-8dc2-2257719f3afa): a small dataset we manually curate at GBIF to override any other list. We mainly use the dataset to add missing names reported by users.
 1. [Catalog of Life](http://www.gbif-uat.org/dataset/7ddf754f-d193-4cc9-b351-99906754a03b): The Catalog of Life provides the entire higher classification above families with the exception of algaes.
 1. [GBIF Algae Classification](http://www.gbif-uat.org/dataset/7ea21580-4f06-469d-995b-3f713fdcc37c): With the withdrawal of Algaebase the current Catalog of Life is lacking any algae taxonomy. To allow other sources to at least provide genus and species names for algae we have created a new dataset that just provides an algae classification down to families. This classification fits right into the empty phyla of the Catalog of Life.

The GBIF portal now also lists [the source datasets that contributed to the GBIF Backbone](http://www.gbif-uat.org/dataset/d7dddbf4-2cf0-4f39-9b2a-bb099caae36c/constituents) and the number of names that were used as primary references.


### Other Improvments
Many occurrence could not be matched to a backbone species in case the name existed multiple times as an accepted taxon. In the new backbone only one version of a name is ever considered accepted. All others now are flagged as doubtful if they are no synonyms. That avoids many ambiguous no-match issues. For example there are many occurrences of *Hyacinthoides hispanica* in Britain which only show up in the new backbone ([old](http://www.gbif.org/occurrence/795765755) / [new](http://www.gbif-uat.org/occurrence/795765755) occurrence, [old](http://api.gbif.org/v1/species/match?verbose=true&kingdom=plantae&name=Hyacinthoides%20hispanica) / [new](http://api.gbif-uat.org/v1/species/match?verbose=true&kingdom=plantae&name=Hyacinthoides%20hispanica) match). This is best seen in the [map comparison of the nub browser](http://mdoering.github.io/nub-browser/app/#/taxon/5304257), try to swipe the map! 

## Known problems
We are aware of some problems with the new backbone which we like to address in the [next stage](http://dev.gbif.org/issues/browse/POR-3029). Two of these issues we consider as candidates for blocking the release of the new backbone:

### Species matching service ignores authorship
As we better keep different authors apart the backbone now contains a lot more species names which just differ by their authorship. The current algorithm only keeps one of these names as the accepted name from the most trusted source (e.g. CoL) and treats the other as doubtful if they are not already treated as synonyms.  

The problem currently is that the species matching service we use to align occurrences to the backbone does [not deal with authorship](http://dev.gbif.org/issues/browse/POR-2768). Therefore we have some cases where occurrences are attached to a doubtful name or even split across some of the "homonyms". 

### Too eager basionym merging
The same epithet is sometimes used by the same author for different names in the same family. This currently leads to an [overly eager basionym grouping](http://dev.gbif.org/issues/browse/POR-2989) with less accepted names.

As these names are still in the backbone and occurrences can be matched to them this is currently not considered a blocker.
