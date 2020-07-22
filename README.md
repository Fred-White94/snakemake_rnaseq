# RNA-seq analysis pipeline

[![Snakemake](https://img.shields.io/badge/snakemake-≥5.2.0-brightgreen.svg)](https://snakemake.bitbucket.io)    
[![Miniconda](https://img.shields.io/badge/miniconda-blue.svg)](https://conda.io/miniconda)

<!-- MarkdownTOC autolink="true" levels="1,2" -->

- [Description](#description)
	- [Description](#description-1)
	- [Input files](#input-files)
	- [Output files](#output-files)
	- [Prerequisites: what you should know before using this pipeline](#prerequisites-what-you-should-know-before-using-this-pipeline)
	- [Content of this GitHub repository](#content-of-this-github-repository)
	- [Pipeline dependencies](#pipeline-dependencies)
- [Installation](#installation)
	- [Download or clone the GitHub repository](#download-or-clone-the-github-repository)
- [Usage \(local machine\)](#usage-local-machine)
	- [:round_pushpin: Option 1: conda \(easiest\)](#round_pushpin-option-1-conda-easiest)
	- [Option 2: Docker :whale: \(recommended\)](#option-2-docker-whale-recommended)
	- [Configuration](#configuration)
	- [Snakemake execution](#snakemake-execution)
	- [Dry run](#dry-run)
	- [Real run](#real-run)
- [Usage \(HPC cluster\)](#usage-hpc-cluster)
- [Directed Acyclic Graph of jobs](#directed-acyclic-graph-of-jobs)
- [References](#references)
	- [Authors](#authors)
	- [Publications](#publications)

<!-- /MarkdownTOC -->


# Description

A Snakemake pipeline for the analysis of _messenger_ RNA-seq data. It processes mRNA-seq fastq files and delivers one raw and one normalised count tables. It can process single or paired-end data and is mostly suited for Illumina sequencing data. 

## Description
This pipeline analyses the raw RNA-seq data and produces two files containing the raw and normalized counts. 

1. The raw fastq files will be trimmed for adaptors and quality checked with `fastp`.  
2. The genome sequence FASTA file will be used for the mapping step of the trimmed reads using `STAR`. 
3. A GTF annotation file will be used to obtain the raw counts using `subread featureCounts`. 
4. The raw counts will be scaled by a custom R function that implements the `DESeq2` median of ratios method to generate the scaled ("normalized") counts. 

## Input files
* __RNA-seq fastq files__ as listed in the `config/samples.tsv` file.
* __A genomic reference in FASTA format__. For instance, a fasta file containing the 12 chromosomes of tomato (*Solanum lycopersicum*).
* __A genome annotation file in the [GTF format](https://useast.ensembl.org/info/website/upload/gff.html)__. You can convert a GFF annotation file format into GTF with the [gffread program from Cufflinks](http://ccb.jhu.edu/software/stringtie/gff.shtml): `gffread my.gff3 -T -o my.gtf`. :warning: for featureCounts to work, the _feature_ in the GTF file should be `exon` while the _meta-feature_ has to be `transcript_id`. 

Below is an example of a GTF file format. :warning: a real GTF file does not have column names (seqname, source, etc.). Remove all non-data rows. 

| seqname | source | feature | start  | end  | score | strand | frame | attributes |
|-----------|------------|------|------|------|---|---|---|----------------------------------------------------------------------------------------------------|
| SL4.0ch01 | maker_ITAG | CDS  | 279  | 743  | . | + | 0 | transcript_id "Solyc01g004000.1.1"; gene_id "gene:Solyc01g004000.1"; gene_name "Solyc01g004000.1"; |
| SL4.0ch01 | maker_ITAG | exon | 1173 | 1616 | . | + | . | transcript_id "Solyc01g004002.1.1"; gene_id "gene:Solyc01g004002.1"; gene_name "Solyc01g004002.1"; |
| SL4.0ch01 | maker_ITAG | exon | 3793 | 3971 | . | + | . | transcript_id "Solyc01g004002.1.1"; gene_id "gene:Solyc01g004002.1"; gene_name "Solyc01g004002.1"; |

## Output files

* __A table of raw counts__ called `counts.txt`: this table can be used to perform a differential gene expression analysis with `DESeq2`. 
* __A table of DESeq2-normalised counts__ called `scaled_counts.tsv`: this table can be used to perform an Exploratory Data Analysis with a PCA, heatmaps, sample clustering, etc.
* __fastp QC reports__ (one per fastq file).

## Prerequisites: what you should know before using this pipeline
- Some command of the Unix Shell to connect to a remote server where you will execute the pipeline (e.g. SURF Lisa Cluster). You can find a good tutorial from the Software Carpentry Foundation [here](https://swcarpentry.github.io/shell-novice/) and another one from Berlin Bioinformatics [here](http://bioinformatics.mdc-berlin.de/intro2UnixandSGE/unix_for_beginners/README.html).
- Some command of the Unix Shell to transfer datasets to and from a remote server (to transfer sequencing files and retrieve the results/). The Berlin Bioinformatics Unix begginer guide available [here] should be sufficient for that (check the `wget` and `scp` commands).
- An understanding of the steps of a canonical RNA-Seq analysis (trimming, alignment, etc.). You can find some info [here](https://bitesizebio.com/13542/what-everyone-should-know-about-rna-seq/).

## Content of this GitHub repository
- `Snakefile`: a master file that contains the desired outputs and the rules to generate them from the input files.
- `config/samples.tsv`:  a file containing sample names and the paths to the forward and eventually reverse reads (if paired-end). **This file has to be adapted to your sample names before running the pipeline**.
- `config/config.yaml`: the configuration files making the Snakefile adaptable to any input files, genome and parameter for the rules.
- `config/refs/`: a folder containing
  - a genomic reference in fasta format. The `S_lycopersicum_chromosomes.4.00.chrom1.fa` is placed for testing purposes.
  - a GTF annotation file. The `ITAG4.0_gene_models.sub.gtf` for testing purposes.
- `fastq/`: a folder containing subsetted paired-end fastq files used to test locally the pipeline. Generated using [Seqtk](https://github.com/lh3/seqtk):
`seqtk sample -s100 {inputfile(can be gzipped)} 250000 > {output(always gunzipped)}`
This folder should contain the `fastq` of the paired-end RNA-seq data, you want to run.
- `envs/`: a folder containing the environments needed for the conda package manager. If run with the `--use-conda` command, Snakemake will install the necessary softwares and packages using the conda environment files.
- `Dockerfile`: a Docker file used to build the docker image that, once run using `docker run rnaseq:dockerfile Snakemake --cores N --use-conda` will trigger installation of the necessary softwares and run the Snakemake pipeline.  

## Pipeline dependencies
* [Snakemake](https://snakemake.readthedocs.io/en/stable/)
* [fastp](https://github.com/OpenGene/fastp)
* [STAR](https://github.com/alexdobin/STAR)   
* [subread](http://subread.sourceforge.net/)  
* [DESeq2](https://bioconductor.org/packages/release/bioc/html/DESeq2.html)


# Installation

## Download or clone the GitHub repository
You will need a local copy of the `snakemake_rnaseq` on your machine. You can either:
- use git in the shell: `git clone git@github.com:BleekerLab/snakemake_rnaseq.git`.
- click on ["Clone or download"](https://github.com/BleekerLab/snakemake_rnaseq/archive/master.zip) and select `download`. 
- Then navigate inside the `snakemake_rnaseq` folder using Shell commands.

# Usage (local machine)

## :round_pushpin: Option 1: conda (easiest)
Using the conda package manager, you need to create an environment where core softwares such as `Snakemake` will be installed.   
1. Install the [Miniconda3 distribution (>= Python 3.7 version)](https://docs.conda.io/en/latest/miniconda.html) for your OS (Windows, Linux or Mac OS X).  
2. Inside a Shell window (command line interface), create a virtual environment named `rnaseq` using the `envs/environment.yaml` file with the following command: `conda env create --name rnaseq --file envs/environment.yaml`
3. Then, before you run the Snakemake pipeline, activate this virtual environment with `source activate rnaseq`.

While a `conda` environment will in most cases work fine, Docker is the recommended solution as it increases pipeline execution reproducibility.

## Option 2: Docker :whale: (recommended)
:round_pushpin: Option 2: using a Docker container  
1. Install Docker desktop for your operating system.
2. Open a Shell window and type: `docker pull bleekerlab/snakemake_rnaseq:4.7.12` to retrieve a Docker image that includes the pipeline required softwares (Snakemake and conda and many others).
3. Run the pipeline on your system with:
`docker run --rm -v $PWD:/home/snakemake/ bleekerlab/snakemake_rnaseq:4.7.12` and add any options for snakemake (`-n`, `--cores 10`) etc.  

This will link your working directory (`$PWD`) to a directory called `/home/snakemake/` inside the container. Results will be written to a folder named `$PWD/results/` (you can change `results` to something you like in the `result_dir` parameter of the `config.yaml`).

The image was built using a [Dockerfile](envs/Dockerfile) based on the [4.7.12 Miniconda3 official Docker image](https://hub.docker.com/r/continuumio/miniconda3/tags).  



## Configuration 
:pencil2: :clipboard:  
You'll need to change a few things to accomodate this pipeline to your needs.
Make sure you have changed the parameters in the `config/config.yaml` file that specifies where to find the sample data file, the genomic and transcriptomic reference fasta files to use and the parameters for certains rules etc.  
This file is used so the `Snakefile` does not need to be changed when locations or parameters need to be changed.

## Snakemake execution
The Snakemake pipeline/workflow management system reads a master file (often called `Snakefile`) to list the steps to be executed and defining their order. It has many rich features. Read more [here](https://snakemake.readthedocs.io/en/stable/).

## Dry run
From the folder containing the `Snakefile`, use the command `snakemake --use-conda -np` to perform a dry run that prints out the rules and commands.

## Real run
Simply type `Snakemake --use-conda` and provide the number of cores with `--cores 10` for ten cores for instance.  
For cluster execution, please refer to the [Snakemake reference](https://snakemake.readthedocs.io/en/stable/executable.html#cluster-execution).

# Usage (HPC cluster)
To be written.


# Directed Acyclic Graph of jobs
![dag](./dag.png)

# References

## Authors
- Marc Galland, m.galland@uva.nl 
- Tijs Bliek, m.bliek@uva.nl
- Frans van der Kloet f.m.vanderkloet@uva.nl

## Publications
- Snakemake publication
- STAR publication
- fastp publication
- subread publication
