# Daily build mechanisms for preventing bad branches

When a large and easily defined category of sequences or dubious mutations are repeatedly causing
bad branches, it saves a lot of work to filter out that category or mask the dubious mutations as
part of the daily build process.
The daily build process has several mechanisms for this:

- [Filtering out sequences using columns of nextclade's TSV output](#filtering-out-sequences-using-nextclade-outputs)
- [Maintaining a file of manually excluded sequence identifiers](#manually-excluding-sequences)
  for sequences that have repeatedly caused problems that require manual fixes
- [Masking out all mutations at positions](#masking-positions-in-the-problematic-sites-set) 
  in the [Problematic Sites](https://github.com/W-L/ProblematicSites_SARS-CoV2)
  set before placing new sequences
- [Branch-specific masking](#branch-specific-masking) of sites or specific mutations after 
  sequences are placed

## Filtering out sequences using nextclade outputs

The sudden emergence of the Omicron variant caused a lot of problems with amplicon dropout
because some primer-binding regions of the genome, especially in Spike, were mutated.
Genome assembly pipelines that inserted reference sequence instead of N bases in low-coverage
regions produced genome sequences that appeared to be Omicron with many reversions to reference,
or in worse cases non-Omicron with a suspiciously high number of Omicron mutations.
That category of sequences caused so much trouble that I added filters to the daily build,
excluding sequences that meet any of these conditions:

1. nextclade's clade assignment is Omicron but there are > 5 reversions to reference
1. nextclade's clade assignment is pre-Omicron but there are > 5 Omicron-associated mutations
1. the total number of substitutions is much less than expected given the number of months since
   December 2019

The first two filters are implemented in the script
[findDropoutContam.pl](../daily-pipeline/scripts.md#finddropoutcontampl)
and the last filter is implemented in
[findRefBackfill.pl](../daily-pipeline/scripts.md#findrefbackfillpl).
Both of those scripts are run by
[makeNewMaskedMaple.sh](../daily-pipeline/scripts.md#makenewmaskedmaplesh).

The filters reduced the amount of bad branches that clustered near the real root of Omicron, but
they also exclude legitimate recombinant sequences that do have more than 5 "reversions" relative
to their assigned clade due to recombination with a sequence outside of that clade.
Therefore we needed a mechanism to exempt some sequences from filtering; the file
[includeRecombinants.tsv](https://github.com/ucscGenomeBrowser/kent/blob/master/src/hg/utils/otto/sarscov2phylo/includeRecombinants.tsv)
lists IDs of sequences that were requested by lineage hunters, and eventually there were so many
requests that Xu Zou made another community-managed file of IDs to exempt from filtering,
[recombinants.tsv](https://raw.githubusercontent.com/sars-cov-2-variants/lineage-proposals/main/recombinants.tsv)
in the repository sars-cov-2-variants/lineage-proposals.

## Manually excluding sequences

Some sequences passed the above filters but still repeatedly caused bad branches when added back
to the tree after the manual fix of pruning and reoptimization (see [Fixing problems manually](manual-fixes.md)).  When I noticed that the same bad branch kept returning, I collected the IDs
of sequences near the base of the branch and added them to a local file, `badBranchSeed.ids`.
That file is used by the script
[makeNewMaskedMaple.sh](../daily-pipeline/scripts.md#makenewmaskedmaplesh) to exclude those
sequences from the list of sequences to be added to the tree.

## Masking positions in the Problematic Sites set

Within the first several months of the SARS-CoV-2 pandemic, several groups recognized that 
certain positions seemed especially prone to either recurrent mutations or sequencing error 
([Issues with SARS-CoV-2 sequencing data, virological.org 2020](https://virological.org/t/issues-with-sars-cov-2-sequencing-data/473)).  
These positions were collected in github repo 
[W-L/Problematic_Sites](https://github.com/W-L/ProblematicSites_SARS-CoV2).
There are two categories of positions: `mask` and `caution`.  Since daily updates commenced in early 2021, we have masked the positions in the `mask` category by filtering mutations at those positions out of the input to UShER, so mutations at those positions do not appear in the tree at all and have no effect on construction of the tree.

Unfortunately there are some positions in the Problematic Sites set that mask Spike amino acid changes, for example position 21575 where a highly homoplasic C>T mutation causes S:L5F and position 21987 where G>A/C/T causes S:G142D/A/V.  The Delta variant includes G21987A
but amplicon dropout was so common that the position was added to the Problematic Sites set,
so the entire UShER tree contains no mutations at position 21987.

## Branch-specific masking

Often, incorrect "reversions" at a particular genomic position affect only part of the tree, for
example only Delta or only BA.1.  Also, we may wish to mask reversions but not mask subsequent 
mutations to other bases, a rare occurrence but interesting when it happens.
The `matUtils mask --mask-mutations` feature enables a more nuanced kind of masking
than we did for Problematic Sites.
The input file has one to three columns: a nucleotide mutation to mask, and then optionally a 
node identifier for the branch in which to mask the mutation, possibly followed by a list of
descendant node identifiers at which to suppress masking so those branches are exempted.
When the mutation has an 'N' instead of an A, C, G or T, any matching mutation is masked.

The daily build process constructs an input file using all three columns.
Since node identifiers must be specified, and node identifiers change every time the tree is
modified, the input file is constructed dynamically from a more stable specification file,
[branchSpecificMask.yml](https://github.com/ucscGenomeBrowser/kent/blob/master/src/hg/utils/otto/sarscov2phylo/branchSpecificMask.yml),
by the script
[branchSpecificMask.py](../daily-pipeline/scripts.md#branchspecificmaskpy).
Comments in branchSpecificMask.yml often refer to GitHub issues or manual inspections that
motivated the addition of sites or mutations to be masked -- helpful information when a
lineage hunter asks, months or years later, why a site is masked.

Unfortunately the branch-specific suppression of reversions causes trouble for recombinant 
lineages: real reversions from recombination may be masked along with undesirable false 
reversions.  So the UShER tree after branch-specific masking, for many recombinant lineages, 
incorrectly includes some mutations that the recombinant lineage does not have.
Also, sometimes real reversions are transmitted and even form new lineage branches.
The third column of the `matUtils mask --mask-mutations` input file was added relatively
recently (December 2025) to address this by allowing descendant nodes to be excluded from
masking.
A corresponding `exclusions` keyword was added to branchSpecificMask.yml and branchSpecificMask.py
to make use of the new third column.
