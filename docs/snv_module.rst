
.. _module_snv_calling:


Module: Single Nucleotide Variant Analysis
===================================================


The Single Nucleotide Variant (SNV) module proceeds in two steps:

#. Single-sample variant calling, with ``run_snps`` command;
#. Population SNV calling across all the samples, with ``merge_snps`` command.

The first step can be run in parallel.
We presuppose users have already completed the :ref:`species selection<module_single_species_selection>`
and have ``midas2_output/sample1/species/species_profile.tsv`` ready for each sample.
Alternatively, advanced users can pass a *prebuilt rep-genome index* (:ref:) ready for single-sample SNV analysis.


.. _species_for_genotype:

Species for Genotype
********************

In a standard SNV/CNV workflow, only **sufficiently abundant species** in the restricted species profile
will be included to build representative genome (rep-genome) or pan-genome index and further to be genotyped.
In order to use the species marker genes profiles to select species for index building in the ``run_snps`` and ``run_genes`` commands, we need to pass
flags specifying the following parameters:

- ``--select_by`` followed by a comma separated list of column names in ``midas2_output/species/species_profile.tsv``
- ``--select_threshold`` followed by a comma-separated list of threshold values for selection.


For most analyses we recommend using the combination of ``median_marker_coverage > 2X`` and ``unique_fraction_covered > 0.5``:

.. code-block:: shell

  --select_by median_marker_coverage,unique_fraction_covered --select_threshold=2,0.5


Some users may wish to genotype low abundant species and should adjust the parameters accordingly:

.. code-block:: shell

    --select_by median_marker_coverage,unique_fraction_covered --select_threshold=0,0.5


Alternatively, users can directly pick a list of species using the ``--species_list`` option.
It is worth noting that the species in the provided species list are still subject to
the ``--select_threshold`` restriction. Users can set ``--select_threshold=-1`` to
escape species selection filters based on the species profiling:

.. code-block:: shell

    --species_list 102337,102506 --select_threshold=-1


**All** the species passing the above mentioned filters will be genotyped in either single-sample SNV or single-sample CNV module.


Population SNV Calling
*************************

Typically, the ``run_snps`` command proceeds by

#.  selecting species based on taxonomic marker gene profiles;
#.  building a sample-specific representative genome (rep-genome) index;
#.  mapping reads with bowtie2 to this index;
#.  outputting read mapping summary and pileup result on a per-species basis.


MIDAS 2.0 purposely holds any filter or species selection upon the single-sample pileup results until across-samples SNV analysis.


Example Command
---------------

In this document, we will keep using the :ref:`example data<example_data>` from Quickstart.


A typical call to ``run_snps`` for one sample is:

.. code-block:: shell

  midas2 run_snps \
    --sample_name sample1 \
    -1 reads/sample1_R1.fastq.gz \
    --midasdb_name uhgg \
    --midasdb_dir my_midasdb_uhgg \
    --select_by median_marker_coverage,unique_fraction_covered \
    --select_threshold=2,0.5 \
    --num_cores 8 \
    midas2_output

.. note::

  The first time ``run_snps`` is used, MIDAS will automatically download
  the reference genomes for the selected species.

.. tip::

   This step can be parallelized over samples (e.g. using shell background
   processes).

.. warning::

   (Race condition) If starting multiple calls to ``run_snps``
   simultaneously, be sure that reference genomes have already been
   downloaded.
   Otherwise multiple redundant downloads may be started.
   TODO: Link to the preload instructions here.

Having run all samples in this way, users next can perform the population SNV
analysis using the ``merge_snps`` command with the default filters:

.. code-block:: shell

    midas2 merge_snps \
      --samples_list list_of_samples.tsv \
      --midasdb_name uhgg \
      --midasdb_dir my_midasdb_uhgg \
      --num_cores 8 \
      midas2_output/merge


Expected Output
---------------

.. _single_sample_snv_summary:

Single-Sample
+++++++++++++

**snps_summary.tsv**

This file ``midas2_output/samples1/snps/snps_summary.tsv`` reports read alignment and pileup summary for all the species in the rep-genome index.

.. csv-table::
  :align: left

  *species_id*,*genome_length*,*covered_bases*,*total_depth*,*aligned_reads*,*mapped_reads*,*fraction_covered*,*mean_coverage*
  102506,5339468,2373275,8045342,468667,224553,0.444,3.390
  102337,2749621,2566404,47723458,1479479,1010530, 0.933,18.595

-   ``species_id``: six-digit species id
-   ``genome_length``: genome length
-   ``covered_bases``: number of bases covered by at least one post-filtered reads
-   ``total_depth``: total read depth across all ``covered_bases``
-   ``aligned_reads``: total read counts across ``covered_bases`` before post-alignment filter
-   ``mapped_reads``: total read counts across ``covered_bases`` after post-alignment filter
-   ``fraction_covered``: fraction of ``covered_bases`` (aka horizontal genome coverage)
-   ``mean_coverage``: mean read depth across all ``covered_bases`` (aka vertical genome coverage)


**Per-species Read Pileup**

This file ``midas2_output/samples1/snps/102506.snps.tsv.lz4`` reports the per-species read pileup for all the genomic sites covered by at least two post-filered reads.

.. csv-table::
  :align: left

  *ref_id*,*ref_pos*,*ref_allele*,*depth*,*count_a*,*count_c*,*count_g*,*count_t*
  gnl|Prokka|UHGG144544_1,881435,T,11,0,0,0,11
  gnl|Prokka|UHGG144544_1,881436,T,13,0,5,0,8
  gnl|Prokka|UHGG144544_1,881437,T,12,0,6,0,6

-   ``ref_id``: scaffold/contig id
-   ``ref_pos``: reference position
-   ``ref_allele``: reference nucleotide
-   ``depth``: number of post-filtered reads
-   ``count_a``: post-filtered read counts of A allele
-   ``count_c``: post-filtered read counts of C allele
-   ``count_g``: post-filtered read counts of G allele
-   ``count_t``: post-filtered read counts of T allele


Across-Samples
+++++++++++++++

**snps_summary.tsv**

This file ``midas2_output/merge/snps/snps_summary.tsv`` merge all single-sample pileup summary for all the species in the :ref:`single-sample pileup summary<single_sample_snv_summary>`.
The reported columns ``genome_length``:``mean_coverage`` are the same with single-sample SNV summary.


.. csv-table::
  :align: left

  *sample_name*,*species_id*,*genome_length*,*covered_bases*,*total_depth*,*aligned_reads*,*mapped_reads*,*fraction_covered*,*mean_coverage*
  sample1,100122,2560878,2108551,10782066,248700,207047,0.823,5.113
  sample2,100122,2560878,2300193,39263110,1180505,820736,0.898,17.069

-  ``sample_name``: unique sample name
-  ``species_id``: six-digit species id


**Per-species SNPs Info File**

This file ``midas2_output/merge/snps/102506.snps_info.tsv.lz4`` reports the population SNV's metadata.

.. csv-table::
  :align: left

    *site_id*,*major_allele*,*minor_allele*,*sample_counts*,*snp_type*,*rc_A*,*rc_C*,*rc_G*,*rc_T*,*sc_A*,*sc_C*,*sc_G*,*sc_T*,*locus_type*,*gene_id*,*site_type*,*amino_acids*
    gnl|Prokka|UHGG000587_14|34360|A,A,C,2,bi,26,10,0,0,2,2,0,0,CDS,UHGG000587_02083,4D,T\\,T\\,T\\,T
    gnl|Prokka|UHGG000587_11|83994|T,G,T,2,bi,0,0,11,45,0,0,2,2,IGR,None,None,None

-  ``site_id``: unique site id, composed of ``ref_id|ref_pos|ref_allele``
-  ``major_allele``: most common/prevalent allele in metagenomes
-  ``minor_allele``: second most common/prevalent allele in metagenomes
-  ``sample_counts``: number of relevant samples where metagenomes is found
-  ``snp_type``: the number of alleles observed at site (mono,bi,tri,quad)
-  ``rc_A``: accumulated read counts of A allele in metagenomes
-  ``rc_C``: accumulated read counts of C allele in metagenomes
-  ``rc_G``: accumulated read counts of G allele in metagenomes
-  ``rc_T``: accumulated read counts of T allele in metagenomes
-  ``sc_A``: accumulated sample counts of A allele in metagenomes
-  ``sc_C``: accumulated sample counts of C allele in metagenomes
-  ``sc_G``: accumulated sample counts of G allele in metagenomes
-  ``sc_T``: accumulated sample counts of T allele in metagenomes
-  ``locus_type``: CDS (site in coding gene), RNA (site in non-coding gene), IGR (site in intergenic region)
-   ``gene_id``: gene identified if locus type is CDS, or RNA
-   ``site_type``: indicates degeneracy: 1D, 2D, 3D, 4D
-   ``amino_acids``: amino acids encoded by 4 possible alleles


**Per-species SNPs Freq Matrix**

This file ``midas2_output/merge/snps/102506.snps_freq.tsv.lz4`` reports site-by-sample allele frequency matrix of population minor allele.

.. csv-table::
  :align: left

  *site_id*,*sample1*,*sample2*
  gnl|Prokka|UHGG000587_11|83994|T,0.692,0.837
  gnl|Prokka|UHGG000587_14|34360|A,0.300,0.269


**Per-species SNPs Depth Matrix**

This file ``midas2_output/merge/snps/102506.snps_freq.tsv.lz4`` reports site-by-sample site depth matrix.
Only accounts for the alleles matching the population major and/or minor allele.

.. csv-table::
  :align: left

  *site_id*,*sample1*,*sample2*
  gnl|Prokka|UHGG000587_11|83994|T,13,43
  gnl|Prokka|UHGG000587_14|34360|A,10,26


Advanced SNV Calling
********************

Adjust Single-Sample Post-alignment Filter
------------------------------------------

Users can adjust post-alignment filters via the following command-line options (default values indicated):

- ``--mapq >= 20``: discard read alignment with alignment quality < 20
- ``--mapid >= 0.94``: discard read alignment with alignment identity < 0.94
- ``--aln_readq >= 20``: discard read alignment with mean quality < 20
- ``--aln_cov >= 0.75``: discard read alignment with alignment coverage < 0.75
- ``--aln_baseq >= 30``: discard bases with quality < 30
- ``--paired_only``: only recruit properly aligned read pairs for post-alignment filter and pileup
- ``--fragment_length 5000``: maximum fragment length for paired-end alignment. Incorrect fragment length would affect the number of proper-aligned read pairs


.. code-block:: shell

    midas2 run_snps
      --sample_name sample1 \
      -1 reads/sample1_R1.fastq.gz \
      --midasdb_name uhgg \
      --midasdb_dir my_midasdb_uhgg \
      --select_by median_marker_coverage,unique_fraction_covered \
      --select_threshold=2,0.5 \
      --fragment_length 3000 --paired_only \
      --num_cores 8 \
      midas2_output


Single-Sample Advanced SNV Calling
----------------------------------

In recognition of the need for single-sample variant calling, we provided ``--advanced`` option to users for single-sample variant calling for all the species in the rep-genome index
with ``run_snps`` command.

In the ``--advanced`` mode, per-species pileup results will also report major allele and minor allele for all the genomic sites covered by at least two post-filtered reads,
upon which custom variant calling filter can be applied by the users.
Users are advised to use the setting ``--ignore_ambiguous`` to avoid falsely calling major/minor alleles for sites with tied read counts.

.. code-block:: shell

    midas2 run_snps
      --sample_name sample1 \
      -1 reads/sample1_R1.fastq.gz \
      --midasdb_name uhgg \
      --midasdb_dir my_midasdb_uhgg \
      --select_by median_marker_coverage,unique_fraction_covered \
      --select_threshold=2,0.5 \
      --fragment_length 2000 --paired_only \
      --advanced --ignore_ambiguous \
      --num_cores 8
      midas2_output


Expected Output
+++++++++++++++

In the ``--advanced`` mode, per-species pileup results will include five additional columns of the major/minor allele for all the covered genomic sites.

.. csv-table::
  :align: left

    *ref_id*,*ref_pos*,*ref_allele*,*depth*,*count_a*,*count_c*,*count_g*,*count_t*,*major_allele*,*minor_allele*,*major_allele_freq*,*minor_allele_freq*,*allele_counts*
    gnl|Prokka|UHGG144544_1,881435,T,11,0,0,0,11,T,T,1.000,0.000,1
    gnl|Prokka|UHGG144544_1,881436,T,13,0,5,0,8,T,C,0.615,0.385,2
    gnl|Prokka|UHGG144544_1,881437,T,12,0,6,0,6,C,T,0.500,0.500,2

-   ``major_allele``: the allele with the most read counts
-   ``minor_allele``: the allele with the 2nd most read counts; same with major_allele if only one allele is observed
-   ``major_allele_freq``: allele frequency of ``major_allele``
-   ``minor_allele_freq``: allele frequency of ``minor_allele``; 0.0 if only one allele is observed
-   ``allele_counts``: number of alleles observed


Adjust Population SNV Filters
-----------------------------

Advanced users can refer to :ref:`this page<population_snv_calling>` for understanding the compute of population SNV.
The species, sample, and site filters for the across-samples SNV calling can be customized with command-line options. For example,

-   We can select species with ``horizontal coverage > 40%``, ``vertical coverage > 3X`` and present in more than 30 relevant samples:

.. code-block:: shell

    --genome_coverage 0.4 --genome_depth 3 --sample_counts 30

-   We can apply the following site selections: only consider site with ``read depth >= 5``, and ``read depth <= 3 * genome_depth``, and the minimal allele frequency to call an allele present is 0.05.

.. code-block:: shell

    --site_depth 5 --site_ratio 3 --snp_maf 0.05

-   We can only report populations SNV meeting the following criteria: bi-allelic, common population SNV (present in more than 80% of the population) from the protein coding genes based on accumulated sample counts.

.. code-block:: shell

    --snp_type bi --snv_type common --site_prev 0.8 --locus_type CDS --snp_pooled_method prevalence

Now we can put all the above-mentioned filters in one `merge_snps` command:

.. code-block:: shell

    midas2 merge_snps
      --samples_list list_of_samples.tsv \
      --midasdb_name uhgg \
      --midasdb_dir my_midasdb_uhgg \
      --genome_coverage 0.4 --genome_depth 3 --sample_counts 30 \
      --site_depth 5 --site_ratio 3 --snp_maf 0.05 \
      --snp_type bi --snv_type common --site_prev 0.8 --locus_type CDS --snp_pooled_method prevalence \
      --num_cores 8 \
      midas2_output/merge
