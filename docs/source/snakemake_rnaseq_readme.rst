RNA-seq analysis pipeline
=========================

| |Snakemake|
| |Miniconda|

.. raw:: html

   <!-- MarkdownTOC autolink="true" levels="1,2" -->

-  `Description <#description>`__

   -  `Description <#description-1>`__
   -  `Input files <#input-files>`__
   -  `Output files <#output-files>`__
   -  `Prerequisites: what you should know before using this
      pipeline <#prerequisites-what-you-should-know-before-using-this-pipeline>`__
   -  `Content of this GitHub
      repository <#content-of-this-github-repository>`__

-  `Installation and usage (local
   machine) <#installation-and-usage-local-machine>`__

   -  `Installation <#installation>`__
   -  `Usage <#usage>`__
   -  `Configuration :pencil2: <#configuration-pencil2>`__
   -  `Dry run <#dry-run>`__
   -  `Real run <#real-run>`__

-  `Installation and usage (HPC
   cluster) <#installation-and-usage-hpc-cluster>`__

   -  `Installation <#installation-1>`__
   -  `Usage <#usage-1>`__

-  `Directed Acyclic Graph of jobs <#directed-acyclic-graph-of-jobs>`__
-  `References :green\_book: <#references-green_book>`__

   -  `Authors <#authors>`__
   -  `Pipeline dependencies <#pipeline-dependencies>`__
   -  `Acknowledgments :clap: <#acknowledgments-clap>`__

.. raw:: html

   <!-- /MarkdownTOC -->

Description
===========

| A Snakemake pipeline for the analysis of *messenger* RNA-seq data. It
processes mRNA-seq fastq files and delivers both raw and
normalised/scaled count tables. This pipeline also outputs a QC report
per fastq file and a ``.bam`` mapping file to use with a genome browser
for instance.
| This pipeline can process single or paired-end data and is mostly
suited for Illumina sequencing data.

Description
-----------

This pipeline analyses the raw RNA-seq data and produces two files
containing the raw and normalized counts.

1. The raw fastq files will be trimmed for adaptors and quality checked
   with ``fastp``.
2. The genome sequence FASTA file will be used for the mapping step of
   the trimmed reads using ``STAR``.
3. A GTF annotation file will be used to obtain the raw counts using
   ``subread featureCounts``.
4. The raw counts will be scaled by a custom R function that implements
   the ``DESeq2`` median of ratios method to generate the scaled
   ("normalized") counts.

Input files
-----------

-  **RNA-seq fastq files** as listed in the ``config/samples.tsv`` file.
   Specify a sample name (e.g. "Sample\_A") in the ``sample`` column and
   the paths to the forward read (``fq1``) and to the reverse read
   (``fq2``). If you have single-end reads, leave the ``fq2`` column
   empty.
-  **A genomic reference in FASTA format**. For instance, a fasta file
   containing the 12 chromosomes of tomato (*Solanum lycopersicum*).
-  **A genome annotation file in the `GTF
   format <https://useast.ensembl.org/info/website/upload/gff.html>`__**.
   You can convert a GFF annotation file format into GTF with the
   `gffread program from
   Cufflinks <http://ccb.jhu.edu/software/stringtie/gff.shtml>`__:
   ``gffread my.gff3 -T -o my.gtf``. :warning: for featureCounts to
   work, the *feature* in the GTF file should be ``exon`` while the
   *meta-feature* has to be ``transcript_id``.

Below is an example of a GTF file format. :warning: a real GTF file does
not have column names (seqname, source, etc.). Remove all non-data rows.

+-------------+---------------+-----------+---------+--------+---------+----------+---------+---------------------------------------------------------------------------------------------------------+
| seqname     | source        | feature   | start   | end    | score   | strand   | frame   | attributes                                                                                              |
+=============+===============+===========+=========+========+=========+==========+=========+=========================================================================================================+
| SL4.0ch01   | maker\_ITAG   | CDS       | 279     | 743    | .       | +        | 0       | transcript\_id "Solyc01g004000.1.1"; gene\_id "gene:Solyc01g004000.1"; gene\_name "Solyc01g004000.1";   |
+-------------+---------------+-----------+---------+--------+---------+----------+---------+---------------------------------------------------------------------------------------------------------+
| SL4.0ch01   | maker\_ITAG   | exon      | 1173    | 1616   | .       | +        | .       | transcript\_id "Solyc01g004002.1.1"; gene\_id "gene:Solyc01g004002.1"; gene\_name "Solyc01g004002.1";   |
+-------------+---------------+-----------+---------+--------+---------+----------+---------+---------------------------------------------------------------------------------------------------------+
| SL4.0ch01   | maker\_ITAG   | exon      | 3793    | 3971   | .       | +        | .       | transcript\_id "Solyc01g004002.1.1"; gene\_id "gene:Solyc01g004002.1"; gene\_name "Solyc01g004002.1";   |
+-------------+---------------+-----------+---------+--------+---------+----------+---------+---------------------------------------------------------------------------------------------------------+

Output files
------------

-  **A table of raw counts** called ``raw_counts.txt``: this table can
   be used to perform a differential gene expression analysis with
   ``DESeq2``.
-  **A table of DESeq2-normalised counts** called ``scaled_counts.tsv``:
   this table can be used to perform an Exploratory Data Analysis with a
   PCA, heatmaps, sample clustering, etc.
-  **fastp QC reports**: one per fastq file.
-  **bam files**: one per fastq file (or pair of fastq files).

Prerequisites: what you should know before using this pipeline
--------------------------------------------------------------

-  Some command of the Unix Shell to connect to a remote server where
   you will execute the pipeline. You can find a good tutorial from the
   Software Carpentry Foundation
   `here <https://swcarpentry.github.io/shell-novice/>`__ and another
   one from Berlin Bioinformatics
   `here <http://bioinformatics.mdc-berlin.de/intro2UnixandSGE/unix_for_beginners/README.html>`__.
-  Some command of the Unix Shell to transfer datasets to and from a
   remote server (to transfer sequencing files and retrieve the
   results/). The Berlin Bioinformatics Unix begginer guide available
   `here <http://bioinformatics.mdc-berlin.de/intro2UnixandSGE/unix_for_beginners/README.html>`__)
   should be sufficient for that (check the ``wget`` and ``scp``
   commands).
-  An understanding of the steps of a canonical RNA-Seq analysis
   (trimming, alignment, etc.). You can find some info
   `here <https://bitesizebio.com/13542/what-everyone-should-know-about-rna-seq/>`__.

Content of this GitHub repository
---------------------------------

-  ``Snakefile``: a master file that contains the desired outputs and
   the rules to generate them from the input files.
-  ``config/samples.tsv``: a file containing sample names and the paths
   to the forward and eventually reverse reads (if paired-end). **This
   file has to be adapted to your sample names before running the
   pipeline**.
-  ``config/config.yaml``: the configuration files making the Snakefile
   adaptable to any input files, genome and parameter for the rules.
-  ``config/refs/``: a folder containing
-  a genomic reference in fasta format. The
   ``S_lycopersicum_chromosomes.4.00.chrom1.fa`` is placed for testing
   purposes.
-  a GTF annotation file. The ``ITAG4.0_gene_models.sub.gtf`` for
   testing purposes.
-  ``.fastq/``: a (hidden) folder containing subsetted paired-end fastq
   files used to test locally the pipeline. Generated using
   `Seqtk <https://github.com/lh3/seqtk>`__:
   ``seqtk sample -s100 <inputfile> 250000 > <output file>`` This folder
   should contain the ``fastq`` of the paired-end RNA-seq data, you want
   to run.
-  ``envs/``: a folder containing the environments needed for the
   pipeline:
-  The ``environment.yaml`` is used by the conda package manager to
   create a working environment (see below).
-  The ``Dockerfile`` is a Docker file used to build the docker image by
   refering to the ``environment.yaml`` (see below).

Installation and usage (local machine)
======================================

Installation
------------

You will need a local copy of the GitHub ``snakemake_rnaseq`` repository
on your machine. You can either: - use git in the shell:
``git clone git@github.com:BleekerLab/snakemake_rnaseq.git``. - click on
`"Clone or
download" <https://github.com/BleekerLab/snakemake_rnaseq/archive/master.zip>`__
and select ``download``. - Then navigate inside the ``snakemake_rnaseq``
folder using Shell commands.

Usage
-----

Configuration :pencil2:
-----------------------

| You'll need to change a few things to accomodate this pipeline to your
needs. Make sure you have changed the parameters in the
``config/config.yaml`` file that specifies where to find the sample data
file, the genomic and transcriptomic reference fasta files to use and
the parameters for certains rules etc.
| This file is used so the ``Snakefile`` does not need to be changed
when locations or parameters need to be changed.

:round\_pushpin: Option 1: conda (easiest)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| Using the conda package manager, you need to create an environment
where core softwares such as ``Snakemake`` will be installed.
| 1. Install the `Miniconda3 distribution (>= Python 3.7
version) <https://docs.conda.io/en/latest/miniconda.html>`__ for your OS
(Windows, Linux or Mac OS X).
| 2. Inside a Shell window (command line interface), create a virtual
environment named ``rnaseq`` using the ``envs/environment.yaml`` file
with the following command:
``conda env create --name rnaseq --file envs/environment.yaml`` 3. Then,
before you run the Snakemake pipeline, activate this virtual environment
with ``source activate rnaseq``.

While a ``conda`` environment will in most cases work just fine, Docker
is the recommended solution as it increases pipeline execution
reproducibility.

:whale: Option 2: Docker (recommended)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| :round\_pushpin: Option 2: using a Docker container
| 1. Install Docker desktop for your operating system. 2. Open a Shell
window and type: ``docker pull bleekerlab/snakemake_rnaseq:4.7.12`` to
retrieve a Docker image that includes the pipeline required softwares
(Snakemake and conda and many others). 3. Run the pipeline on your
system with:
``docker run --rm -v $PWD:/home/snakemake/ bleekerlab/snakemake_rnaseq:4.7.12``
and add any options for snakemake (``-n``, ``--cores 10``) etc. The
image was built using a `Dockerfile <envs/Dockerfile>`__ based on the
`4.7.12 Miniconda3 official Docker
image <https://hub.docker.com/r/continuumio/miniconda3/tags>`__.

:whale: Option 3: Singularity
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Install singularity
2. Open a Shell window and type:
   ``singularity run docker://bleekerlab/snakemake_rnaseq:4.7.12`` to
   retrieve a Docker image that includes the pipeline required software
   (Snakemake and conda and many others).
3. Run the pipeline on your system with
   ``singularity run snakemake_rnaseq_4.7.12.sif`` and add any options
   for snakemake (``-n``, ``--cores 10``) etc. The directory where the
   sif file is stored will automatically be mapped to
   ``/home/snakemake``. Results will be written to a folder named
   ``$PWD/results/`` (you can change ``results`` to something you like
   in the ``result_dir`` parameter of the ``config.yaml``).

Dry run
-------

-  With conda: use the ``snakemake -np`` to perform a dry run that
   prints out the rules and commands.
-  With Docker: use the ``docker run``

Real run
--------

With conda: ``snakemake --cores 10``

Installation and usage (HPC cluster)
====================================

Installation
------------

You will need a local copy of the GitHub ``snakemake_rnaseq`` repository
on your machine. On a HPC system, you will have to clone it using the
Shell command-line:
``git clone git@github.com:BleekerLab/snakemake_rnaseq.git``. - click on
`"Clone or
download" <https://github.com/BleekerLab/snakemake_rnaseq/archive/master.zip>`__
and select ``download``. - Then navigate inside the ``snakemake_rnaseq``
folder using Shell commands.

Usage
-----

See the detailed protocol `here <./hpc/README.md>`__.

Directed Acyclic Graph of jobs
==============================

.. figure:: ./dag.png
   :alt: dag

   dag
References :green\_book:
========================

Authors
-------

-  Marc Galland, m.galland@uva.nl
-  Tijs Bliek, m.bliek@uva.nl
-  Frans van der Kloet f.m.vanderkloet@uva.nl

Pipeline dependencies
---------------------

-  `Snakemake <https://snakemake.readthedocs.io/en/stable/>`__
-  `fastp <https://github.com/OpenGene/fastp>`__
-  `STAR <https://github.com/alexdobin/STAR>`__
-  `subread <http://subread.sourceforge.net/>`__
-  `DESeq2 <https://bioconductor.org/packages/release/bioc/html/DESeq2.html>`__

Acknowledgments :clap:
----------------------

`Johannes Köster <https://johanneskoester.bitbucket.io/>`__; creator of
Snakemake.

.. |Snakemake| image:: https://img.shields.io/badge/snakemake-≥5.2.0-brightgreen.svg
   :target: https://snakemake.bitbucket.io
.. |Miniconda| image:: https://img.shields.io/badge/miniconda-blue.svg
   :target: https://conda.io/miniconda
