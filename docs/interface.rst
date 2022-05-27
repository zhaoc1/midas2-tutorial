MIDAS 2.0 Interface
===================

.. contents::
   :depth: 3


.. _common_cli_options:

Common CLI Options
******************

TODO: output dir, sample name, database dir/name,

Output Directory
----------------

MIDAS 2.0 writes its outputs to a user-specified root directory,
which is always passed as a mandatory argument to each of the MIDAS 2.0 analyses command.

For example, in :ref:`Quickstart<demo_midas_ourdir>`, ``midas2_output`` is the chosen output directory, and all analyses steps operate within it.


Single-sample Commands
----------------------

The three single-sample commands (``run_species``, ```run_snps`` and ``run_genes``) share a number of command-line flags.

Sample Name
+++++++++++

Users need to chose a unique ``sample_name`` per sample, and together with the output directory,
``midas2_output/sample_name1`` constitutes the unique output directory for single-sample analyses.


Input Reads
+++++++++++

The FASTA/FASTQ file containing single-end or paired-ends sequencing reads needs to be passed via the arguments as:

.. code-block:: shell

    --R1 /path/to/R1.fastq.gz  --R2 /path/to/R2.fastq.gz


Across-samples Commands
-----------------------

A tab-separated sample manifest file listing the ``sample_name`` and full path of the single-sample root output directory
``midas_output`` is required for across-samples analyses. And users need to pass the path of this file to the command-line argument ``--sample_list``.
For example, in the Quickstart, we passed as following: ``--sample_list list_of_samples.tsv``.


A template is shown here:

.. csv-table::
  :align: left

  *sample_name*,*midas_outdir*
  sample1,/home/ubuntu/MIDAS2.0/tests/midas2_output
  sample2,/home/ubuntu/MIDAS2.0/tests/midas2_output


MIDASDB
-------

For all MIDAS 2.0 analysis, users need to choose

#. a valid precomputed MIDASDB name (uhgg, gtdb) as ``--midasdb_name``
#. a valid local path for the downloaded MIDASDB ``--midasdb_dir``.

For example, in :ref:`QuickStart<example_data>`, we downloaded the SCG marker database for ``--midasdb_name uhgg`` into
``--midasdb_dir my_midasdb_uhgg``.


Others Parameters
-----------------

Users can set the ``--num_cores`` to the number of physical cores to use: e.g. ``--num_cores 16``.

And all MIDAS 2.0 analyses can print out the full help message and exit by ``-h`` or ``--help``.
