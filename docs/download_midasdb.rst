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

If we the following list of species ids (here an example with only two species) to a plain text file named `species.list`: ::

  $ echo -e "100078\n102478" > species_list.txt

we can then run the following to preload all of the data needed for these two species:

.. code-block:: shell

    MIDAS 2.0 database \
        --download
        --midasdb_name uhgg \
        --midasdb_dir my_midasdb_uhgg \
        --species_list species.list


Advanced users may be interested in downloading species present in a list of samples,
:ref:`refer to this page <database_download>`.

It is also possible for advance users to :doc:`contruct their own MIDASDB <build_your_own>`
from a custom genome collection (e.g. for metagenome assembled genomes).
