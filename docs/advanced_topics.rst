
Advanced Topics
===============

.. contents::
   :depth: 3


.. _population_snv_calling:

Population SNV Calling
**********************

In this section, we will the compute of population SNV, the chunkified implementation of pileup, as well as filters of species, samples and genomic sites in the **SNV** module.

Important Concepts
------------------

#.  **<species, relevant samples> selection**

    Population SNV analysis restricts attention to "sufficiently well" covered species in "sufficiently many" samples.

    To be specific, a given <species, sample> pair will only be kept (by default) if it has more than 40% horizontal genome coverage (``genome_coverage``)
    and 5X vertical genome coverage (``genome_depth``).
    Furthermore, only "sufficiently prevalent" species with "sufficiently many" (``sample_counts``) would be included for the population SNV analysis.
    Therefore, different species may have different lists of relevant samples.

#.  **<site, relevant samples> selection**

    For each genomic site, a sample is considered to be "relevant" if the corresponding site depth falls between the range defined by the input
    arguments ``site_depth`` and ```site_ratio * mean_genome_coverage``; otherwise it is ignored for the across-samples SNV compute.

    Therefore, different genomic sites from the same species may have different panels of "relevant samples".
    And genomic site prevalence can be computed as the ratio of the number of relevant samples for the given site over the total number of relevant samples for the given species.

#.  **relevant site**

    For each species, a site is considered to be "relevant" if the site prevalence meets the range defined by the input arguments ``snv_type`` and ```site_prev``.
    By default, common SNV with more than 90% prevalence are reported.


.. _population_snv_computation:

Population SNV Computation
--------------------------

There are three main steps to compute and report population SNV in MIDAS 2.0.

First, for each relevant genomic site, MIDAS 2.0 determines the set of alleles present across all relevant samples.
Specifically, for each allele (A, C, G, T), ``merge_snps`` command

#. tallys the sample counts (``sc_``) of relevant samples containing corresponding allele (``scA:scT``)
#. sums up the read counts (``rc`_`) of the corresponding allele across all the relevant samples (``rc_G:rc_T``).

.. csv-table::
  :align: left

    site_id,rc_A,rc_C,rc_G,rc_T,sc_A,sc_C,sc_G,sc_T
    gnl|Prokka|UHGG000587_14|34360|A,26,10,0,0,1,2,0,0

Second, population major and minor alleles for a single site can be computed based on the
accumulated read counts or sample counts across all relevant samples.
The population major allele refers to the most abundant/prevalent allele, and
the population minor allele refers to the second most prevalent/abundant allele.

For example, the population major allele of the site ``gnl|Prokka|UHGG000587_14|34360|A`` in the above example is ``A`` defined
by accumulated read counts and ``C`` defined by accumulated sample counts.

Third, MIDAS 2.0 collects and reports the sample-by-site matrix of the corresponding (1) site depth and (2)
allele frequency of the above calculated population minor allele for all the relevant samples.
In these two matrices, MIDAS 2.0 encode ``site_depth = 0`` and ``allele_frequency = -1`` with the special meaning of missing <site, sample> pair.


Chunkified Pileup Implementation
--------------------------------

Both single-sample and across-samples pileup are parallelized on the unit of chunk of sites, which is indexed by <species_id, chunk_id>.
Only when all chunks from the same species finished processing, chunk-level pileup results will merged into species-level pileup result.

This implementation makes population SNV analysis across thousands of samples possible.
To compute the population SNV for one chunk, all the pileup results of corresponding sites across all the samples need to be read into memory.
With the uses of multiple CPUs, multiple chunks can be processed at the same time.
Therefore, for large collections of samples, we recommend higher CPU counts and smaller chunk size to
optimally balance memory and I/O usage, especially for highly prevalent species.
Users can adjust the number of sites per chunk via ``chunk_size`` (default value = 1000000).
MIDAS 2.0 also has a ``robust_chunk`` option, where assigning different chunk sizes to different species based on the species prevalence.
