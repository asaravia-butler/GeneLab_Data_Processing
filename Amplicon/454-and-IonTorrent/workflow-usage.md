# Workflow information and usage instructions


## General workflow info
The processing is implemented as a [Snakemake](https://snakemake.readthedocs.io/en/stable/) workflow and utilizes [conda](https://docs.conda.io/en/latest/) environments. The workflow can be used even if unfamiliar with Snakemake and conda, but if wanting to learn more about those, [this Snakemake tutorial](https://snakemake.readthedocs.io/en/stable/tutorial/tutorial.html) within [Snakemake's documentation](https://snakemake.readthedocs.io/en/stable/) is a good place to start for that, and an introduction to conda with installation help and links to other resources can be found [here at Happy Belly Bioinformatics](https://astrobiomike.github.io/unix/conda-intro).  

## Utilizing the workflow

1. [Installing conda and snakemake](#installing-conda-and-snakemake)  
2. [Downloading the workflow template files](#downloading-the-workflow-template-files)  
3. [Modifying the variables in the config.yaml file](#modifying-the-variables-in-the-configyaml-file)  
4. [Running the workflow](#running-the-workflow)  

### 1. Installing conda and snakemake
We recommend installing a Miniconda, Python3 version appropriate for your system, as exemplified in [the above link](https://astrobiomike.github.io/unix/conda-intro#getting-and-installing-conda).  

Once we have conda, we can install the latest snakemake like so:

```bash
conda install -c conda-forge -c bioconda -c defaults snakemake
```

### 2. Downloading the workflow template files
All files required for utilizing the workflow are in the [workflow-template](workflow-template) directory. To get a copy of that directory on to our system, we can copy the github web address of that directory, paste it into [GitZip here](http://kinolien.github.io/gitzip/), and then click download.


### 3. Modifying the variables in the config.yaml file
We can then modify the variables in the [config.yaml](workflow-template/config.yaml) file as needed, like pointing to the directory where our starting reads are located.

### 4. Running the workflow

Here is one example command of how we can run the workflow from within the directory holding the Snakefile, config.yaml, and other workflow files we downloaded:

```bash
snakemake --use-conda --conda-prefix ${CONDA_PREFIX}/envs -j 2 -p
```

* `--use-conda` – this specifies to use the conda environments included in the workflow (these are specified in the [envs](envs) directory
* `--conda-prefix` – this allows us to point to where the needed conda environments should be stored. Including this means if we use the workflow on a different dataset somewhere else in the future, it will re-use the same conda environments rather than make new ones. The value listed here, `${CONDA_PREFIX}/envs`, exactly like that, is the default location for conda environments (the variable `${CONDA_PREFIX}` will be expanded to the appropriate location on whichever system it is run on).
* `-j` – this lets us set how many jobs Snakemake should run concurrently (keep in mind that many of the thread and cpu parameters set in the config.yaml file will be multiplied by this)
* `-p` – specifies to print out each command being run to the screen

See `snakemake -h` and [Snakemake's documentation](https://snakemake.readthedocs.io/en/stable/) for more options and details.

---
