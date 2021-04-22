Getting Started
###############

First - ‘Preparing Your Account’ and ‘Miniconda’ installation instructions. All the necessary information about the pipeline is in https://github.com/BleekerLab/snakemake_rnaseq. 


* Go to your personal directory

.. code-block::	shell

	cd ~/personal

* Download the preprocessing pipeline from github

.. code-block::	shell

	git clone https://github.com/BleekerLab/snakemake_rnaseq

* Go to snakemake folder

.. code-block::	shell

	cd snakemake_rnaseq/




Installation
++++++++++++

Install mamba to install the environment faster (than using conda)
.. code-block::	shell

	conda install -c conda-forge mamba

Create the environment
.. code-block::	shell

	mamba env create --name rnaseq --file environment.yaml














To adjust the pipeline to your experiment, you need to modify some parameters. 


For this you can either use a text editor for example `gedit <https://wiki.gnome.org/Apps/Gedit>`_ or you can download it using an ssh client like `Filezilla <https://filezilla-project.org/>`_ and open it on your own machine. 

The former is recommended. If you do the latter, make sure to upload the modified file and replace it.

* Reference Genome and Annotation gtf files

In the ‘config/refs’ folder, add the reference genome and the annotation file of your choice. Then change the name of the files in the ‘config.yaml’ file, adding the path to it. eg.:

.. code-block::	yaml

	refs: 
	    genome:  "config/refs/hg38.analysisSet.fa"
	    gtf:     "config/refs/gencode.v37.annotation.gtf"


* Sample file

Make sure to replace the example config/samples.tsv file with the one relevant to your own experiment. This should include at least 3 columns: sample, fq1 and fq2.




