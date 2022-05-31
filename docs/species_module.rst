Module: Species Selection
==================================


Reference-based metagenotyping depends crucially on the choice of reference sequences and
incorrect mapping of reads is a major problem. Microbiome data usually contains hundreds
of species in one sample, and an ideal reference database is both representative and
comprehensive in terms of the species in the sample. A badly chosen reference may suffer
both from ambiguous mapping of reads to two or more sequences or spurious mapping to
incorrect sequences. Therefore, a typical MIDAS 2.0 workflow starts with a species selection step,
which filters the MIDASDB to sufficiently abundant species in each particular
sample.


.. _module_single_species_selection:

Single-Sample Species Profiling
**********************************

Species coverage is estimated via profiling 15 universal single copy marker genes (SCGs), to
quickly detect the panel of abundant species in the sample.

.. warning::

  This is designed *only* to select species with sufficient coverage in each
  sample. It is not intended to quantify species abundance.


Example Command
---------------

In this document, we will keep using the :ref:`example data<example_data>` from Quickstart.

.. code-block:: shell


  MIDAS 2.0 run_species \
      --sample_name sample1 \
      -1 reads/sample1_R1.fastq.gz \
      --midasdb_name uhgg \
      --midasdb_dir my_midasdb_uhgg \
      --num_cores 4 \
      midas2_output

.. note::

  The first time ``run_species`` is used, MIDAS will automatically download
  the marker gene database.

.. note::

   This step can be parallelized over samples (e.g. using shell background
   processes)

.. warning::

   (Race condition) If starting multiple calls to ``run_species``
   simultaneously, be sure that the marker gene database has already been
   downloaded.
   Otherwise multiple redundant downloads may be started.
   TODO: Link to the preload instructions here.


Expected Output
---------------

species_profile.tsv
+++++++++++++++++++

The primary output of ``run_species`` command is ``midas2_output/samples1/species/species_profile.tsv`` which
describes the coverage of each species' marker genes in the sample.
Species are sorted in decreasing order of ``median_marker_coverage``.
Only species with more than two marker genes covered with more than two reads (a very low bar) are reported.

.. csv-table::
  :align: left

  *species_id*,*marker_read_counts*,*median_marker_coverage*,*marker_coverage*,*marker_relative_abundance*,*unique_fraction_covered*
  102337,4110,28.48,28.91,0.30,1.00
  102506,734,4.98,4.98,0.05,0.93

-   ``species_id``: six-digit species id
-   ``marker_read_counts``: total mapped read counts
-   ``median_marker_coverage``: median coverage of the 15 SCGs
-   ``marker_coverage``: mean coverage of the 15 SCGs
-   ``marker_relative_abundance``: computed based on ``marker_coverage``
-   ``unique_fraction_covered``: the fraction of uniquely mapped SCGs genes

We will later use the ``median_marker_coverage`` and ``unique_fraction_covered``
to select **sufficiently abundant species** for the downstream single-smaple SNV or CNV analysis.


Across-Samples Species Merging
******************************

Example Command
---------------

We can run ``merge_species`` command to merge all the single-sample species profiling
result for all the samples listed in the :ref:`samples_list<prepare_sample_list>`.

.. code-block:: shell

  midas2 merge_species \
    --samples_list list_of_samples.tsv \
    --midasdb_name uhgg \
    --midasdb_dir my_midasdb_uhgg \
    --num_cores 4 \
    --min_cov 2 \
    midas2_output/merge

- ``--min_cov``: minimal ``median_marker_coverage`` for estimating species prevalence ``sample_counts``.


Expected Output
---------------

.. _species_prevalence:

species_prevalence.tsv
++++++++++++++++++++++

The primary output of the across-samples species merging analysis is the file ``midas2_output/merge/species/species_prevalence.tsv``.

.. csv-table::
  :align: left

  *species_id*,*median_abundance*,*mean_abundance*,*median_coverage*,*mean_coverage*,*sample_counts*
  102337,0.186,0.186,16.205,16.205,2
  102506,0.035,0.035,2.967,2.967,2

-   ``species_id``: six-digit species id
-   ``median_abundance``: median ``marker_relative_abundance`` across samples
-   ``mean_abundance``: average ``marker_relative_abundance`` across samples
-   ``median_coverage``: median ``median_marker_coverge`` across samples
-   ``mean_coverage``: average ``median_marker_coverge`` across samples
-   ``sample_counts``: number of samples with ``median_marker_coverge >= min_cov``


**Species-by-sample Matrix**

MIDAS 2.0 reports a few species-by-sample matrix that can be found at: ``midas2_output/merge/species``.

- Species-by-sample median marker coverage matrix is located at ``midas2_output/merge/species/species_marker_median_coverage.tsv``.

.. csv-table::
  :align: left

  *species_id*,*sample1*,*sample2*
  102337,3.926,28.484
  102506,0.951,4.983

-  Species-by-sample unique fraction covered matrix is located at ``midas2_output/merge/species/species_unique_fraction_covered.tsv``.

.. csv-table::
  :align: left

  *species_id*,*sample1*,*sample2*
  102337, 1,1
  102506,0.92,1


.. _database_download:

Download Database For Selected Species
**************************************


List of Species
---------------

We can compile one comprehensive list of species across samples in the same study.
For example, we can get the list of species that is present in at least one sample:

.. code-block:: shell

  awk '$6 > 1 {print $6}' midas2_output/merge/species/species_prevalence.tsv > all_species_list.tsv


Download MIDASDB
----------------
..
    TODO: Remove this section; just link to the relevant instructions in the
    Download MIDASDB page as a `tip`.

We can then download the MIDASDB only for species in the ``all_species_list.tsv``:

.. code-block:: shell

  midas2 database --download \
    --midasdb_name uhgg \
    --midasdb_dir my_midasdb_uhgg \
    --species_list my_species_list.tsv


Having finished the species selection step, we can now go to the SNV or CNV
modules, depending on the scientific aims.
