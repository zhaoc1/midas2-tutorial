Quickstart
============

.. contents::
   :depth: 3


Setup
*****


Install MIDAS 2.0
-----------------


Pre-download SCG Genes
**********************

Download the universal single copy genes for MIDAS reference database (MIDAS DB) of ``uhgg``
to a new folder called ``my_midasdb_uhgg`` ::

  $ midas2 database --download --midasdb_name uhgg --midasdb_dir my_midasdb_uhgg



Run SNV Module
**************

Single-Sample SNV Calling
-------------------------

We first run the single-sample SNV analysis for each sample.
The pileup summary for ``sample1`` will be generated under the directory
``midas2_output/sample1/snps/snps_summary.tsv``, which summarize the read mapping
and pileup results of the species in the restricted species profile
``median_marker_coverage > 2``.

.. code-block:: shell

  for sample_name in sample1 sample2
  do
    midas2.0 run_snps \
      --sample_name ${sample_name} \
      -1 reads/${sample_name}_R1.fastq.gz \
      --midasdb_name uhgg \
      --midasdb_dir my_midasdb_uhgg \
      --num_cores 4 \
      midas2_output
  done


DONE
++++
