# Getting Started

The GitHub repository 
[SARS-CoV-2-tree-maintenance](https://github.com/AngieHinrichs/SARS-CoV-2-tree-maintenance)
holds the record of manual tree fixes: dated session logs in
[notes/](https://github.com/AngieHinrichs/SARS-CoV-2-tree-maintenance/tree/main/notes),
the [summary.tsv](https://github.com/AngieHinrichs/SARS-CoV-2-tree-maintenance/blob/main/summary.tsv)
index, and this documentation.

The scripts that actually run the daily build live in a different
repository: the UCSC kent repo, hosted at UCSC with a mirror on GitHub at
[ucscgenomebrowser/kent](https://github.com/ucscgenomebrowser/kent).
See [Daily Update Pipeline](daily-pipeline/overview.md) for a guide to the scripts.

You will need a local clone of both of those repositories.  Here is one way to fetch them
from GitHub:

```bash
mkdir -p ~/github
cd ~/github
git clone git@github.com:AngieHinrichs/SARS-CoV-2-tree-maintenance.git
git clone git@github.com:ucscGenomeBrowser/kent.git
```

## Prerequisites

Tools referenced throughout the notes and docs:

- **[UShER](https://github.com/yatisht/usher)** — provides `usher`,
  `matUtils`, and `matOptimize`, used for placing samples, extracting/
  pruning/masking, and re-optimizing the mutation-annotated tree.
  The daily update pipeline scripts expect a local build from source of usher-sampled and
  matUtils but the `usher` package may also be installed using
  [BioConda](https://bioconda.github.io/).
- **[Nextclade](https://github.com/nextstrain/nextclade)** — clade
  assignment and QC metrics used for filtering (see
  [Fixing problems via daily build mechanisms](manual-curation/build-fixes.md)). See
  [Nextclade CLI installation instructions](https://docs.nextstrain.org/projects/nextclade/en/stable/user/nextclade-cli/installation/index.html).
- **[Taxonium](https://taxonium.org/)** / `usher_to_taxonium` — for
  converting a `.pb` tree to a Taxonium file for visual inspection. A file from a local
  clone of [theosanderson/taxonium](https://github.com/theosanderson/taxonium)
  is used in some workflows (see [manual-fixes.md](manual-curation/manual-fixes.md)).
  ```
  pip install taxoniumtools

  cd ~/github
  git clone git@github.com:theosanderson/taxonium.git  
  ```
- **Taxonium** app installation - for viewing a very large tree locally.
  See [Installing the Taxonium app](manual-curation/taxonium-inspection.md#installing-the-taxonium-app)
- Python 3 and Perl — some of the daily-build helper scripts under
  `src/hg/utils/otto/sarscov2phylo/` (e.g. `branchSpecificMask.py`,
  `findDropoutContam.pl`, `findRefBackfill.pl`) are written in one or the
  other.

!!! note
    The daily build combines public sequence data with data from GISAID's
    EpiCoV™ database. A tree maintainer must have
    access to GISAID's EpiCoV™ database; if you don't already have access,
    apply for it at [gisaid.org/register](https://gisaid.org/register).

## SARS-CoV-2-tree-maintenance repository layout

```text
SARS-CoV-2-tree-maintenance/
├── notes/          # dated logs of manual tree-fixing sessions
├── summary.tsv     # index of all manual fixing sessions
├── docs/           # this documentation site
├── mkdocs.yml      # config for this documentation site
└── README.md
```

## Setting up your environment

Most of the build scripts set the environment variables they need for
themselves, near the top of the script. The same variables are also useful
in an interactive shell when following along with a manual fix (see
[Fixing Problems Manually](manual-curation/manual-fixes.md)), so it's worth
setting them in your `~/.bashrc` too:

| Variable | Points to |
|----------|-----------|
| `$ottoDir` | A directory on a **very large disk**. This is where daily builds accumulate, along with downloads from NCBI, COG-UK, and CNCB. Daily builds have been accumulating at UCSC since early 2021; as of August 2026, even after removing and compressing some intermediate files, they take up 38 TB. |
| `$scriptDir` | The path to `kent/src/hg/utils/otto/sarscov2phylo` in your clone of the kent repo. |
| `$usherDir` | The path to your local clone of [yatisht/usher](https://github.com/yatisht/usher), if you're using a locally compiled build of the UShER tools rather than a system install. |
| `$gisaidDir` | A directory on a large disk (much less space required than `$ottoDir`) for GISAID downloads. |
| `$ncbiDir` | `$ottoDir/ncbi.latest`, a symbolic link (kept up to date by `getNcbi.sh`) pointing to the latest successful daily download from NCBI. |

## Where to go next

- Running the automated updates → [Daily Update Pipeline](daily-pipeline/overview.md)
- Fixing a reported tree problem → [Manual Curation](manual-curation/overview.md)
- Annotating a new Pango lineage on the the tree → [Annotating Lineages](annotation/overview.md)
