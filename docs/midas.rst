MIDAS 2.0 Overview
==================

Metagenomic Intra-Species Diversity Analysis System (`MIDAS <https://genome.cshlp.org/content/26/11/1612>`_)
is an integrated set of workflows for **profiling strain-level genomic variations in shotgun metagenomic data**.
The original MIDAS harnesses a reference database of 5,926 species (MIDAS DB v1.2).


MIDAS 2.0 was developed to address the computational challenges presented by increasingly large reference genome databases.
MIDAS 2.0 implements the same analyses as the original `MIDAS tool <https://github.com/snayfach/MIDAS>`_,
but re-engineered to

#. host multiple MIDAS Reference Database (MIDAS DB) from public microbial genome collections
#. enable thousands of metagenomic samples to be efficiently genotyped.


MIDAS 2.0 contains two strain-level reads-to-table analysis modules:

#. Population single nucleotide variant (SNV) analyses: :ref:`SNV module <module_snv_calling>`
#. Population pan-gene copy number variation (CNV) analyses: :ref:`CNV module <module_cnv_calling>`


Each module includes two sequential steps: single-sample analysis and across-samples analysis.
