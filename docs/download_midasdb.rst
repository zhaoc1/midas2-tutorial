Download MIDASDB
================

.. contents::
   :depth: 3


List MIDASDB
***************

MIDAS Reference Database (MIDASDB) is comprised of representative genomes,
species pangenomes, and marker genes. For MIDAS 2.0, we pre-build two MIDASDBs from public
microbial genome database:

.. code-block:: shell

  midas2 database --list

  uhgg 286997 genomes from 4644 species version 1.0
  gtdb 258405 genomes from 47893 species version r202


Init MIDASDB
***************

For the purposes of this documentation we'll generally assume that we're working
with the prebuilt ``uhgg`` MIDASDB and that the local mirror is in a subdirectory
``my_midasdb_uhgg``.

We highly recommend that first time users initialize a local MIDASDB. This command
will download the minimal information needed to run the :ref:`abundant species detection <species_detection>`.

.. code-block:: shell

  midas2 database \
    --init \
    --midasdb_name uhgg \
    --midasdb_dir my_midasdb_uhgg


Download All Species
********************

While it is possible to download an entire MIDASDB using the following
command:

.. code-block:: shell

  midas2 database \
    --download \
    --midasdb_name uhgg \
    --midasdb_dir my_midasdb_uhgg \
    --species all


This requires a large amount of data transfer and storage: 93 GB for ``MIDASDB-uhgg``
and 539 GB for ``MIDASDB-gtdb``.

Download Selected Species
*************************

Instead, we recommend that users take advantage of species-level database
sharding to download and decompress only the necessary portions of a
MIDASDB.


For a collection of samples, we can first merge the abundant specie profiling results
across sample:

.. code-block:: shell

  midas2 merge_species \
    --samples_list list_of_samples.tsv \
    --midasdb_name uhgg \
    --midasdb_dir my_midasdb_uhgg \
    --num_cores 4 \
    midas2_output/merge

Second, we can get the list of species that is present in at least one sample:

.. code-block:: shell

  awk '$6 > 1 {print $6}' midas2_output/merge/species/species_prevalence.tsv > all_species_list.tsv


Last, we can download the MIDAS DB only for species in the ``all_species_list.tsv``:

.. code-block:: shell

  midas2 database --download \
    --midasdb_name uhgg \
    --midasdb_dir my_midasdb_uhgg \
    --species_list my_species_list.tsv


It is also possible for advance users to :doc:`contruct their own MIDASDB <build_your_own>`
from a custom genome collection (e.g. for metagenome assembled genomes)
