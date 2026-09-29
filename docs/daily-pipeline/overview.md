# Daily Update Pipeline — Overview

The daily update of the UShER SARS-CoV-2 tree, running since early 2021 while accumulating many
changes since then, is a `cron` job
on a linux server running scripts in the src/hg/utils/otto/sarscov2phylo/ directory of
the [kent](https://github.com/ucscgenomebrowser/kent) repository.  The scripts for the
job are described in the [Scripts Reference](scripts.md).  

`updatePublic.sh` runs on a bare-metal server at UCSC with 256 CPUs and 755GB RAM,
triggered by `cron`. The scripts use only a fraction of that capacity, so
the pipeline is not resource-constrained by the hardware.

The main steps of the build, as of September 2026, are as follows:

* Fetch data from [GISAID](https://gisaid.org), [NCBI](https://ncbi.nlm.nih.gov), 
  [COG-UK](https://www.sanger.ac.uk/collaboration/covid-19-genomics-uk-cog-uk-consortium/) and 
  the China National Center for Bioinformation ([CNCB](https://www.cncb.ac.cn/));
  run pangolin and nextclade on new sequences
  ([gisaidFromChunks.sh](scripts.md#gisaidfromchunkssh),
  [getNcbi.sh](scripts.md#getncbish),
  [getCogUk.sh](scripts.md#getcoguksh),
  [getCncb.sh](scripts.md#getcncbsh))
* Deduplicate sequences submitted to both a public sequence repository (NCBI/ENA/DDBJ, COG-UK, CNCB) and GISAID
  ([updateIdMapping.sh](scripts.md#updateidmappingsh))
* Find sequences from each repository that are not already in the tree from the previous days'
  build
  ([makeNewMaskedMaple.sh](scripts.md#makenewmaskedmaplesh))
* Filter sequences by quality measures (length, number of non-N bases, number of reversions
  relative to nextclade-assigned clade, number of substitutions relative to sampling date)
  ([makeNewMaskedMaple.sh](scripts.md#makenewmaskedmaplesh))
* Convert alignments for sequences that pass the filters into the MAPLE/"diff" format for UShER,
  masking sites in the [Problematic Sites](https://github.com/W-L/ProblematicSites_SARS-CoV2) set
  ([makeNewMaskedMaple.sh](scripts.md#makenewmaskedmaplesh))
* Run usher-sampled on the previous day's tree and the MAPLE/"diff" inputs for new sequences
  ([updateCombinedTree.sh](scripts.md#updatecombinedtreesh))
* Perform branch-specific masking (i.e. remove specific substitutions from specific branches of
  the tree where they have been found to be especially error-prone)
  ([branchSpecificMask.py](scripts.md#branchspecificmaskpy))
* Filter out excessively long branches (note: the notion of "excessively long" must be adjusted
  over time as the virus diverges from the original reference sequence)
  ([updateCombinedTree.sh](scripts.md#updatecombinedtreesh))
* Run matOptimize to improve the tree under maximum parsimony
  ([updateCombinedTree.sh](scripts.md#updatecombinedtreesh))
* Repeat the long-branch filtering
  ([updateCombinedTree.sh](scripts.md#updatecombinedtreesh))
* Combine metadata from GISAID, NCBI, COG-UK and CNCB into a unified TSV file
  ([combineMetadata.sh](scripts.md#combinemetadatash))
* Annotate Nextstrain clades and Pango lineages on nodes of the tree
  ([updateCombinedTree.sh](scripts.md#updatecombinedtreesh))
* Add Nextstrain clade and Pango lineage inferred from placement on the tree to metadata TSV file
  ([updateCombinedTree.sh](scripts.md#updatecombinedtreesh))
* Install files for full tree (including GISAID) in dev.usher.bio area (staging for usher.bio)
  ([updateCombinedTree.sh](scripts.md#updatecombinedtreesh))
* Extract recent-only subtree (samples from past year plus samples within radius of 5 nodes) for
  faster search on usher.bio
  ([updateCombinedTree.sh](scripts.md#updatecombinedtreesh))
* Make memory-mapped hash table files for metadata and name lookup
  ([updateCombinedTree.sh](scripts.md#updatecombinedtreesh))
* Make Taxonium file for viewing/debugging full tree
  ([updateCombinedTree.sh](scripts.md#updatecombinedtreesh))
* Extract public-sequence-only subtree, download files, and stage the files for download and
  (dev.)usher.bio
  ([extractPublicTree.sh](scripts.md#extractpublictreesh))
* Screen the tree for split Pango lineages
  ([updateCombinedTree.sh](scripts.md#updatecombinedtreesh))
* Screen the tree for missing or extra Pango lineage annotations
  ([updateCombinedTree.sh](scripts.md#updatecombinedtreesh))

The email output of the cron job ends with a summary of the number of sequences in the full tree and public tree, and warnings about missing, extra or split Pango lineages if applicable.

See the [Scripts Reference](scripts.md) for details on each script,
its inputs, outputs, and command-line options.
