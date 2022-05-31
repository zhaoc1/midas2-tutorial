
.. _module_cnv_calling:

Module: Copy Number Variant Analysis
=============================================

Similar to the SNV module, the Copy Number Variant (CNV) module also proceeds in two steps:

#. Single-sample pan-genome copy number variants calling, with ``run_genes`` command;
#. Merge these results into a summary across all samples, with ``merge_genes`` command.

The first step can be run in parallel.
We presuppose users have already completed the :ref:`Species Selection<module_single_species_selection>`
and have ``midas2_output/sample1/species/species_profile.tsv`` ready for each sample.
Alternatively, advanced users can pass a *prebuilt rep-genome index* (:ref:) ready for single-sample SNV analysis.

.. contents::
   :depth: 3


Population CNV Calling
**********************

Typically, the ``run_genes`` command proceeds by

#.  selecting species based on taxonomic marker gene profiles;
#.  building a sample-specific pan-genome index;
#.  mapping reads with bowtie2 to this index;
#.  outputting read mapping summary and copy number profiling result on a per-species basis.


Example Command
---------------

In this document, we will keep using the :ref:`example data<example_data>` from Quickstart.

We presuppose users also completed the :ref:`Species for Genotype<species_for_genotype>`. A typical call to ``run_genes`` for one sample is:

.. code-block:: shell

  midas2 run_genes \
    --sample_name sample1 \
    -1 reads/sample1_R1.fastq.gz \
    --midasdb_name uhgg \
    --midasdb_dir my_midasdb_uhgg \
    --select_by median_marker_coverage,unique_fraction_covered \
    --select_threshold=2,0.5 \
    --num_cores 8 \
    midas2_output

Having run the single-sample CNV analysis for all the samples listed in the ``list_of_samples.tsv``,
users next can merge the results and product a summary with `merge_genes` command with default parameters.

.. code-block:: shell

    midas2 merge_genes \
      --samples_list list_of_samples.tsv \
      --midasdb_name uhgg \
      --midasdb_dir my_midasdb_uhgg \
      --num_cores 8 \
      midas2_output/merge

Expected Output
---------------

.. _single_sample_gene_summary:

Single-Sample
+++++++++++++

**genes_summary.tsv**

This file ``midas2_output/samples1/genes/genes_summary.tsv`` reports read alignment and CNV calling summary for all the species in the pan-genome index.

.. csv-table::
  :align: left

   *species_id*,*pangenome_size*,*covered_genes*,*fraction_covered*,*mean_coverage*,*aligned_reads*,*mapped_reads*,*marker_coverage*
   102337,15578,4468,0.287,16.213,1650361,450353,20.213
   102506,731186,4733, 0.006,3.803,681335,37272,2.140

- ``species_id``: six-digit species id
- ``pangenome_size``: number of centroids (non-redundant genes) in the species pangenome
- ``covered_genes``: number of centroids covered with at least one post-filtered read
- ``fraction_covered``: fraction of ``covered_genes`` over ``pangenome_size``
- ``mean_coverage``: average read depth across ``covered_genes``
- ``aligned_reads``: total number of aligned reads before post-alignment filter
- ``mapped_reads``: total number of aligned reads after post-alignment filter
- ``marker_coverage``: average read depth across 15 universal SCGs in the species pangenome


Per species per centroid copy numbers are computed in three steps:

#.  Per centroid, read alignment metrics, e.g ``mapped_reads`` and ``mean_coverage``, are computed;
#.  Per species, median read coverage of all the mapped centroids corresponding to the 15 universal SCGs are identified;
#.  Per centroid, ``copy numbers`` are computed and gene presence/absence are further inferred.


**Per-species Pan-gene CNV Calling**

This file ``midas2_output/samples1/genes/102506.genes.tsv.lz4`` reports the per-species CNV calling for all the pan-genes covered by at least two post-filered reads.

.. csv-table::
  :align: left

   *gene_id*,*gene_length*,*aligned_reads*,*mapped_reads*,*mean_coverage*,*fraction_covered*,*copy_number*
   UHGG143901_00483,555,14,6,2.961538,0.234234,1.384035
   UHGG143901_03589,384,103,57,32.840708,0.294271,15.347667
   UHGG143902_04031,207,9,2,1.737500,0.386473,0.811997

- ``gene_id``: centroid id in the species pan-genome
- ``gene_length``: gene length
- ``aligned_reads``: number of aligned reads to ``gene_id`` before post-alignment filter
- ``mapped_reads``: number of aligned reads to ``gene_id`` after post-alignment filter
- ``mean_coverage``: average read depth of ``gene_id`` based on ``mapped_reads`` (``total_gene_depth / covered_bases``)
- ``fraction_covered``: proportion of the ``gene_id`` covered by at least one read (``covered_bases / gene_length``)
- ``copy_number``: estimated copy number of ``gene_id`` based on ``mapped_reads`` (``mean_coverage / median_marker_coverage``)


Across-Samples
+++++++++++++++

**genes_summary.tsv**

This file ``midas2_output/merge/genes/genes_summary.tsv`` merge all single-sample CNV calling summary for all the species in the :ref:`single-sample results<single_sample_gene_summary>`.
The reported columns ``covered_genes``:``marker_coverage`` are the same with single-sample CNV summary.

.. csv-table::
  :align: left

  *sample_name*,*species_id*,*pangenome_size*,*covered_genes*,*fraction_covered*,*mean_coverage*,*aligned_reads*,*mapped_reads*,*marker_coverage*
  sample1,100122,  29165,,   2535,,   0.087,,, 4.723,,  263395,, 53006,, 1.435
  sample2,100122,  29165,,   3212,,   0.110,,, 16.095,, 1447684,,263878,,10.713

- ``sample_name``: unique sample name
- ``species_id``: six-digit species id


**Per-species Pan-gene CNV Matrix**

This file ``midas2_output/merge/genes/102506.genes_copynum.tsv.lz4`` reports gene-by-sample copy number matrix.

.. csv-table::
  :align: left

  *gene_id*,*sample1*,*sample2*
  UHGG000587_00401,33.969154,19.891455
  UHGG000587_01162,5.703398,2.821237
  UHGG000587_00962,2.370930,0.289325


**Per-species Pan-gene Presence Absence Matrix**

This file ``midas2_output/merge/genes/102506.genes_preabs.tsv.lz4`` reports gene-by-sample presence absence matrix.

.. csv-table::
  :align: left

   *gene_id*,*sample1*,*sample2*
   UHGG000587_00401,1,1
   UHGG000587_01162,1,1
   UHGG000587_00962,1,0


****Per-species Pan-gene Mean Coverage Matrix**

This file ``midas2_output/merge/genes/102506.genes_depth.tsv.lz4`` reports gene-by-sample mean coverage matrix.

.. csv-table::
  :align: left

  *gene_id*,*sample1*,*sample2*
  UHGG000587_00401,48.747945,213.090622
  UHGG000587_01162,8.184746,30.222978
  UHGG000587_00962,3.402439,3.099448


Advanced CNV Calling
********************

Adjust Single-Sample Post-alignment Filter
------------------------------------------

Users can adjust post-alignment quality filter parameters via the command-line options (default vlaues indicated):

-  ``--mapq >= 2``: reads aligned to more than one genomic locations equally well are discarded (MAPQ=0,1)
-  ``--mapid >= 0.94``: discard read alignment with alignment identity < 0.94
-  ``--aln_readq >= 20``: discard read alignment with mean quality < 20
-  ``--aln_cov >= 0.75``: discard read alignment with alignment coverage < 0.75


Adjust Population CNV Filters
-----------------------------

The default ``merge_genes`` results are reported for pan-genes clustered at 95% identity (``cluster_pid``).
It further quantify the presence/absence for pan-genes by comparing the ``copy_number`` with the
user-defined minimal gene copy number (``min_copy``).
``cluster_pid`` and ``min_copy`` can be customized with the following command-line options:

- ``--genome_depth``: filter out species with ``mean_coverage`` < 1X.
- ``--min_copy``: genes with ``copy_number`` >= 0.35 are classified as present.
- ``--cluster_pid``: gene CNV results can be reported at various clustering cutoffs {75, 80, 85, 90, 95, 99}.
