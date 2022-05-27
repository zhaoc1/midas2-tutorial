Quickstart
============

.. contents::
   :depth: 3


Setup
*****


Install MIDAS 2.0
-----------------


On a Linux machine, download a copy of MIDAS 2.0 from our GitHub repository, and
install the dependencies. We do not currently support non-Linux environments.


.. code-block:: shell

  $ git clone https://github.com/czbiohub/MIDAS2.0.git
  $ cd MIDAS2.0

  $ conda env create -n MIDAS 2.0 -f MIDAS 2.0.yml
  $ cpanm Bio::SearchIO::hmmer --force # Temporary fix for Prokka

  $ conda activate midas2.0
  $ pip install .


Alternative installation procedures are detailed in :doc:`this page <installation>`.


.. _example_data:

Example Data
------------

We prepare two single-end gzipped FASTQ files in the folder ``MIDAS2.0/tests/reads``.
Navigate to the ``tests`` directory ::

  $ cd tests


Pre-download SCG Genes
**********************

Download the universal single copy genes for MIDAS reference database (MIDAS DB) of ``uhgg``
to a new folder called ``my_midasdb_uhgg`` ::

  $ midas2 database --download --midasdb_name uhgg --midasdb_dir my_midasdb_uhgg


..
  TODO: Add links to the more completely explanations of each step
  elsewhere in the wiki.


Identify Abundant Species
*************************

.. code-block:: shell

  for sample_name in sample1 sample2
  do
    midas2.0 run_species \
        --sample_name ${sample_name} \
        -1 reads/${sample_name}_R1.fastq.gz \
        --midasdb_name uhgg \
        --midasdb_dir my_midasdb_uhgg \
        --num_cores 4 \
        midas2_output
    done


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


Across-Samples SNV Calling
--------------------------

.. _prepare_sample_list:

Prepare Sample List
+++++++++++++++++++

A tab-separated sample manifest file listing the ``sample_name`` and ``midas_output`` is required for
across-samples analysis.

.. code-block:: shell

  echo -e "sample_name\tmidas_outdir" > list_of_samples.tsv
  ls reads | awk -F '_' '{print $1}' | awk -v OFS='\t' '{print $1, midas2_output}' >> list_of_samples.tsv


We can take a look at the ``list_of_samples.tsv``: ::

  $ cat list_of_samples.tsv


Population SNVs
+++++++++++++++

Now we are ready to compute the population SNVs across the two samples:

.. code-block:: shell

  midas2 merge_snps \
    --samples_list list_of_samples.tsv \
    --midasdb_name uhgg \
    --midasdb_dir my_midasdb_uhgg \
    --genome_coverage 0.7 \
    --num_cores 4 \
    midas2_output/merge


Run CNV Module
**************


Single-Sample CNV Calling
-------------------------

We first run the single-sample CNV analysis for each sample.
The pileup summary for ``sample1`` will be generated under the directory
``midas2_output/sample1/genes/genes_summary.tsv``.

.. code-block:: shell

  for sample_name in sample1 sample2
  do
    midas2.0 run_genes \
      --sample_name ${sample_name} \
      -1 reads/${sample_name}_R1.fastq.gz \
      --midasdb_name uhgg \
      --midasdb_dir my_midasdb_uhgg \
      --num_cores 4 \
      midas2_output
  done


Across-Sample CNV Calling
-------------------------

We can merge the per-sample CNV results:

.. code-block:: shell

  midas2.0 run_genes \
    --samples_list list_of_samples.tsv \
    --midasdb_name uhgg \
    --midasdb_dir my_midasdb_uhgg \
    --num_cores 4 \
    midas2_output/merge
