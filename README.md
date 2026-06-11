# SARS-CoV-2 tree maintenance

This repository contains a record of manual modifications to the daily updated UCSC UShER tree of SARS-CoV-2 genomes.
Details of the commands run to make the changes are in text files in the notes/ directory.  
The file summary.tsv lists the start date, notes file, tree date(s), major types of changes, and comments for each manual fixing session.

The majority of the modifications were simple pruning followed by re-optimization of the tree, and mostly motivated by user complaints that a branch was placed incorrectly, i.e. they would prefer to see the path of mutations from root to the branch in a different order.
Other types of changes include force moves, updates to names of sequences in the tree (before the daily update process did that automatically), and restoring lineage annotations that were missed in the daily update.
