Installation
============

.. contents::
   :depth: 2


MIDAS 2.0 and all its dependencies can be installed in a few ways.

Conda
+++++++++++++

.. code-block:: shell

  conda config --set channel_priority flexible
  conda install -c zhaoc1 -c anaconda -c bioconda -c conda-forge -c defaults MIDAS 2.0


.. tip::

   If this is your first time installing Conda, you'll need to add it to your shell's
   path. If you're running Bash, add the following
   command your path: ``echo 'export
   PATH=$PATH:$HOME/miniconda3/bin' > ~/.bashrc``


From source
++++++++++++

On a Linux machine, download a copy of MIDAS 2.0 from our GitHub repository, and
install MIDAS 2.0 from the source code.


.. code-block:: shell

  $ git clone https://github.com/czbiohub/MIDAS2.0.git
  $ cd MIDAS2.0
  $ conda env create -n MIDAS 2.0 -f MIDAS 2.0.yml
  $ cpanm Bio::SearchIO::hmmer --force # Temporary fix for Prokka
  $ pip install .



Docker
++++++++++++

.. code-block:: shell

  docker pull zhaoc1/MIDAS 2.0:latest
  docker run --volume "/home/ubuntu/.aws":"/root/.aws":ro --rm -it MIDAS 2.0:latest
