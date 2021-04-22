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


