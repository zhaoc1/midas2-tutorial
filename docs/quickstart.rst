Quickstart
============

Setup
*****


Install MIDAS 2.0
-----------------

..
    Does MacOS not work? Maybe just _not_ Windows?

On a Linux machine, download a copy of MIDAS 2.0 from our GitHub repository, and
install the dependencies. We do not currently support non-Linux environments.


..
    Confirmed that this is the best install method? Seems like the most complicated one...
    I guess part of the goal here is to get all of the test reads which are saved to
    github...

.. code-block:: shell

  $ git clone https://github.com/czbiohub/midas2.0.git
  $ cd MIDAS2.0

  $ conda env create -n midas2.0 -f midas2.yml
  $ cpanm Bio::SearchIO::hmmer --force # Temporary fix for Prokka

  $ conda activate midas2.0
  $ pip install .


Alternative installation procedures are also
:doc: `described elsewhere <installation>`_.
However, we'll be using test files kept in this git repository.

.. _example_data:

Example Data
------------

Assuming you're in the ``MIDAS2.0/`` directory you just ``cd``-ed to,
two single-end gzipped FASTQ files are in the folder ``tests/reads``.
Navigate to the ``tests`` directory ::

  $ cd tests


.. _init_db:

Pre-download SCG Genes
**********************

..
    I think you should delete this pre-loading step, since
    MIDAS is designed to do it automatically.
    If you intend to remove this functionality soon, but
    otherwise I think it fits the quickstart mentality to use
    as much of the automated stuff as possible.

Download the universal single copy genes for MIDAS reference database (MIDAS DB) of ``uhgg``
to a new folder called ``my_midasdb_uhgg`` ::

  $ midas2 database --init --midasdb_name uhgg --midasdb_dir my_midasdb_uhgg

..
    TODO: If I'm not mistaken, this will install the MIDASDB to MIDAS2.0/tests/my_midasdb_uhgg
    Seems like a mistake, since users will run the quickstart and then have to _redo_ the
    database download when they want to run MIDAS on different project...
    TODO: Is there a similar issue with using the _installation_ detailed above?
    Will users need to uninstall and re-install for some reason?
    TODO: Add links to the more completely explanations of each step
    elsewhere in the wiki.


.. _demo_midas_ourdir:

Identify Abundant Species
*************************

We'll start by searching for reads that align to single-copy, taxonomic marker
genes in order to identify abundant species in each sample.

.. code-block:: shell

  for sample_name in sample1 sample2
  do
    midas2 run_species \
        --sample_name ${sample_name} \
        -1 reads/${sample_name}_R1.fastq.gz \
        --midasdb_name uhgg \
        --midasdb_dir my_midasdb_uhgg \
        --num_cores 4 \
        midas2_output
    done

..
    TODO: Removing as many of the arguments as possible would be ideal, since this
    is supposed to be the "simplest possible" run. However, the only arg that
    seems removable is num_cores...
    TODO: (Software) Consider renaming --num_cores to --num-cores. The latter
    is the UNIX standard for long option names. For backwards compatibility
    you'll want to leave the underscore form, but most users will expect a
    dash for word breaks in argument names.


Single-nucleotide Variant Analysis
**********************************

Identify SNVs in Each Sample
----------------------------
..
    Is "SNV calling" an accurate description of what MIDAS is doing here?
    Seems more like this step is just about alignment to the reference
    genome and SNV-calling only really happens in the cross-sample analysis.

We'll next run the single-sample SNV analysis for each sample.

.. code-block:: shell

  for sample_name in sample1 sample2
  do
    midas2 run_snps \
      --sample_name ${sample_name} \
      -1 reads/${sample_name}_R1.fastq.gz \
      --midasdb_name uhgg \
      --midasdb_dir my_midasdb_uhgg \
      --num_cores 4 \
      midas2_output
  done

The pileup summary for, ``sample1`` is written to
``midas2_output/sample1/snps/snps_summary.tsv``.
This file summarizes the read mapping
and pileup results for each of the abundant species determined in the previous
step.
By default, species are selected based on the filter:
``median_marker_coverage > 2``.

..
    TODO: Link to detailed information about this filtering.


Compile SNVs across samples
---------------------------

..
    TODO: "Across-samples" is a bit clunky as a descriptor of this step.
    To my ear, something like "cross-sample" or "multi-sample" or writing it
    all the way out as "SNV calling across multiple samples" would be
    the more obvious phrasing.

.. _prepare_sample_list:


In order to combine SNV results from multiple samples, we'll first
construct a tab-separated sample "manifest" file.
This file has a column for the ``sample_name`` and another for
``midas_output``, and is required for multi-sample analyses.

.. code-block:: shell

  echo -e "sample_name\tmidas_outdir" > list_of_samples.tsv
  ls reads | awk -F '_' '{print $1}' | awk -v OFS='\t' '{print $1, "midas2_output"}' >> list_of_samples.tsv


..
    TODO: The shell command to build this file is a bit opaque, and users
    may have other ideas about how to build it. Maybe skip the shell
    script and just provide the manifest already in ``reads/``?

We can take a look at the ``list_of_samples.tsv``:

.. code-block:: shell

  cat list_of_samples.tsv

..
    TODO: What's the output look like? Show readers so they can tell if they
    messed something up in the previous step.


Based on this output, we can run ``merge_snps`` and MIDAS 2.0 will know to
look at ``midas2_output/sample1/snps/snps_summary.tsv`` for the ``run_snps``
output from sample1.

..
    (Software) Is there a reason the user needs to manually construct the path to
    the MIDAS output directories? Seems like just a list of sample names
    and the output directory passed as a command argument should be enough to
    guess the path...
    If there are major use-cases for user-specified output paths then I'm not
    aware of them. Perhaps this should be opt-in...


Now we are ready to compute the population SNVs across the two samples:

.. code-block:: shell

  midas2 merge_snps \
    --samples_list list_of_samples.tsv \
    --midasdb_name uhgg \
    --midasdb_dir my_midasdb_uhgg \
    --genome_coverage 0.7 \
    --num_cores 4 \
    midas2_output/merge

..
    As above, it would be good to remove as many of the unecessary CLI
    options as possible.
    (Software) It would be good to have reasonable defaults for all of these.
    Currently, every MIDAS requires a bunch of identical options every single
    command. This means there are more opportunities for typos and the
    shell history gets messy.

Users may be most interested in the contents of the file
``midas2_output/merge/TODO`` written in this step.

..
    TODO: Quickly show the contents of this most-important output file.

Other output files and the full output directory structure can be found at
:ref:`MIDAS 2.0 Target Layout<target_layout>`.

Copy-number Variant Analysis
**********************************

Identify CNVs in Each Sample
----------------------------

We first run the single-sample CNV analysis for each sample.
The pileup summary for ``sample1`` will be generated under the directory
``midas2_output/sample1/genes/genes_summary.tsv``.

.. code-block:: shell

  for sample_name in sample1 sample2
  do
    midas2 run_genes \
      --sample_name ${sample_name} \
      -1 reads/${sample_name}_R1.fastq.gz \
      --midasdb_name uhgg \
      --midasdb_dir my_midasdb_uhgg \
      --num_cores 4 \
      midas2_output
  done


Compile CNVs across samples
---------------------------

..
    TODO: Point users to the sample manifest built in the previous module.
    If they scipped the SNV analysis, they'll still need to do that step.
    (Consider doing that step entirely separately from the SNV and CNV modules.)

We can merge the per-sample CNV results:

.. code-block:: shell

  midas2 run_genes \
    --samples_list list_of_samples.tsv \
    --midasdb_name uhgg \
    --midasdb_dir my_midasdb_uhgg \
    --num_cores 4 \
    midas2_output/merge

..
  TODO: If you do quickstart correctly, this is the output you will see.


Users may be most interested in the contents of the file
``midas2_output/merge/TODO`` written in this step.

..
    TODO: Quickly show the contents of this most-important output file.

Other output files and the full output directory structure can be found on
the :ref:`Target Layout<target_layout>` documentation.
