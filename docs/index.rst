.. MIDAS2.0 documentation master file, created by
   sphinx-quickstart on Wed May 25 18:30:13 2022.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to MIDAS 2.0
====================================


Metagenomic Intra-Species Diversity Analysis System 2.0 (MIDAS 2.0) is an
integrated pipeline for profiling single nucleotide variants (SNVs) and gene
copy number variants (CNVs) in shotgun metagenomic reads. MIDAS 2.0 implements
the same analyses as the original
`MIDAS <https://github.com/snayfach/MIDAS>`_,
but re-engineered to addresses the computational challenges presented by
increasingly large reference genome databases.

MIDAS 2.0 was developed by `Chunyu Zhao <chunyu.zhao@czbiohub.org>`_
and Boris Dimitrov in the `Pollard Lab <https://docpollard.org/>`_ at
Chan Zuckerberg Biohub.
MIDAS 2.0 expands on the original MIDAS developed by
`Stephen Nayfach <snayfach@gmail.com>`_.

..
    TODO: Link CZB?
    TODO: Replace use of "MIDAS 2.0" where it's obvious you're talking
    about the current software with only "MIDAS". The "2.0" is self-evident
    in most cases (the above paragraphs being one of the few exceptions).

Source code is `available on GitHub
<https://github.com/czbiohub/MIDAS2.0.git>`_.

.. note::

   This project is under active development.

Contents
--------

.. toctree::
    :maxdepth: 1

    quickstart
    installation
    overview
    species_module
    snv_module
    cnv_module
    download_midasdb
    output
    advanced_topics
    glossary
    acknowledgements
