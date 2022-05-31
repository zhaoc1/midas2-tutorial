Installation
============

MIDAS 2.0 and all its dependencies can be installed in a few ways.

Conda
+++++++++++++

We recommend that most users use Conda.

..
    TODO: Is this true?

`Install Conda
<https://conda.io/projects/conda/en/latest/user-guide/install/index.html>`_ if
you have not already. Then,

.. code-block:: shell

  conda config --set channel_priority flexible
  conda install -c zhaoc1 -c anaconda -c bioconda -c conda-forge -c defaults MIDAS 2.0



Docker
++++++++++++

We also provide a pre-built Docker container.

.. code-block:: shell

  docker pull zhaoc1/MIDAS 2.0:latest
  docker run --volume "/home/ubuntu/.aws":"/root/.aws":ro --rm -it MIDAS 2.0:latest



From Source
++++++++++++

Alternatively, users who want the most up-to-date version of the MIDAS code
can install from source (dependencies installed with Conda).

.. code-block:: shell

  $ git clone https://github.com/czbiohub/MIDAS2.0.git
  $ cd MIDAS2.0
  $ conda env create -n MIDAS 2.0 -f MIDAS 2.0.yml
  $ cpanm Bio::SearchIO::hmmer --force # Temporary fix for Prokka
  $ pip install .

.. tip::

    Using the pip ``--editable`` flag here (``pip install --editable .``)
    is useful for those wishing to modify the MIDAS source code directly.

We've also included integration tests, which can be run using the provided
script ::

  $ bash tests/test_analysis.sh 8

This will run an example analysis with 8 cores,
and will verify that all the dependencies are correctly installed
and that all analysis modules of MIDAS 2.0 can run properly.
We recommend running this after installing MIDAS 2.0 from source.
