# SARS-CoV-2 Phylogenetic Tree Maintenance

This documentation covers the tools and procedures used to maintain a
daily updated phylogenetic tree of SARS-CoV-2 genomes. It is split
into several parts:

- **[Daily Update Pipeline](daily-pipeline/overview.md)** — the automated
  scripts that run daily to pull new genome sequences and metadata, place new sequences on the
  tree, and publish the updated tree and metadata.
- **[Manual Curation](manual-curation/overview.md)** — the human-in-the-loop
  process for identifying and fixing problematic branching structures that
  the community flags, using [Taxonium](https://taxonium.org/) for
  inspection and various methods to correct the tree.
- **[Annotating New Pango Lineages](annotation/overview.md)** - how to determine
  which node of the tree corresponds to the lineage designator's intent,
  and update the daily build files to reliably identify the same node every
  day.

<!-- TODO when process is cleaned up
- **[Pangolin Data Release](pangolin-data/overview.md)** - how to make a release
  of [pango-designation](https://github.com/cov-lineages/pango-designation),
  make and test a minimized version of the full SARS-CoV-2 tree for use in the
  [pangolin](https://github.com/cov-lineages/pangolin) tool, and make a
  coordinated release of the
  [pangolin-data](https://github.com/cov-lineages/pangolin-data) and
  [pangolin-assignment](https://github.com/cov-lineages/pangolin-assignment)
  repositories.
-->

If you're new here, start with [Getting Started](getting-started.md) to set
up your environment.

