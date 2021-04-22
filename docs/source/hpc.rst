Slurm and snakemake for Dummies
##############################


To use slurm on cruchomics, first check which nodes are being use and on the next step choose the one which has the least traffic (from cn001 to cn005) 

.. code-block::	shell

	squeue

Follow the salloc instructions to allocate the compute power you specify

.. code-block::	shell

	salloc -N 1 -w omics-cn005 --cpus-per-task 30 --mem=30G

Activate the environment

.. code-block::	shell

	conda activate rnaseq

For a tryout of the pipeline, do a dry run in the snakemake_rnaseq folder

.. code-block::	shell

	snakemake -np 

Run the pipeline specifying the number of cores to be used

.. code-block::	shell

	srun snakemake --cores 30

If the run fails, do this to unlock the directory and try again!

.. code-block::	shell

	srun snakemake -np --unlock

In another terminal window, check if the pipeline is working (make sure the node is the same as in salloc). 

.. code-block::	shell

	srun -n 1  -t 1 --pty -w omics-cn004 bash -i
	top



Check the %CPU and %mem (RAM) usage, the CPU will not use more compute power than you specified with (--cpus-per-task 30 and --cores 30) but the RAM will always try to use what it needs to complete the job, which will cause problems if you do not allocate enough memory (--mem=30G).


To stop the command top (or any other) do ‘control c’



The output files will be in two different folders:

* Temp
In ‘mapped’ - the RNA-Seq read alignment files: *.bam

* Results
Unscaled RNA-Seq read counts: counts.txt
Differential expression file: results.tsv (this is redundant when using the version of the pipeline which counts at the exon level)

*In ‘fastp’ - the fastqc report files: *.html
*In ‘logs’ - the mapping statistics: *sum.txt




