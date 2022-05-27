MIDAS 2.0 Overview
==================

MIDAS 2.0
---------
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


MIDAS Reference Database
------------------------

MIDAS 2.0, as any reference-based strain-level genomic variation analysis pipeline, also presuppose a reference database construction
step has already taken place.
**MIDAS Reference Database (MIDASDB** refers to a set of custom files needed for the strain-level metagenomic analysis.

The original MIDAS provided a default bacterial reference databases (see `Figure 1 <https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5088602/>`_):
`MIDAS DB v1.2 <http://lighthouse.ucsf.edu/MIDAS/midas_db_v1.2.tar.gz>`_ was constructed from a collection of 5952 bacterial species clusters
representing 31,007 high-quality bacterial genome.

In the past few years, the number of sequenced microbial genomes have increased vastly,
in particular with the addition of metagenome-assembled genomes (MAGs) sequenced from varied habitats.
Therefore, it is necessary to update MIDAS 2.0 reference database accordingly.
On the other hand, processing the large amount of available genome sequences poses a significant compute challenge.

For MIDAS 2.0, instead of generating the species clusters from scratch,
we take advantage of several published collections of prokaryotic genome databases, and build new MIDASDB individually.

More information can refer to :doc:`Download MIDASDB <download_midasdb>`.
