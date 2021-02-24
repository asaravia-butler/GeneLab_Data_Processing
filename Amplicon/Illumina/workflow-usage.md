# Workflow information and usage instructions


## General workflow info
The current processing protocol is implemented as a [Snakemake](https://snakemake.readthedocs.io/en/stable/) workflow and utilizes [conda](https://docs.conda.io/en/latest/) environments. The workflow can be used even if you are unfamiliar with Snakemake and conda, but if you want to learn more about those, [this Snakemake tutorial](https://snakemake.readthedocs.io/en/stable/tutorial/tutorial.html) within [Snakemake's documentation](https://snakemake.readthedocs.io/en/stable/) is a good place to start for that, and an introduction to conda with installation help and links to other resources can be found [here at Happy Belly Bioinformatics](https://astrobiomike.github.io/unix/conda-intro).  

## Utilizing the workflow

1. [Install conda and snakemake](#1-install-conda-and-snakemake)  
2. [Download the workflow template files](#2-download-the-workflow-template-files)  
3. [Modify the variables in the config.yaml file](#3-modify-the-variables-in-the-configyaml-file)  
4. [Run the workflow](#4-run-the-workflow)  

### 1. Install conda and snakemake
We recommend installing a Miniconda, Python3 version appropriate for your system, as exemplified in [the above link](https://astrobiomike.github.io/unix/conda-intro#getting-and-installing-conda).  

Once conda is installed on your system, you can install the latest version of snakemake by running the following command:

```bash
conda install -c conda-forge -c bioconda -c defaults snakemake
```

### 2. Download the workflow template files
All files required for utilizing the GeneLab workflow for processing Illumina amplicon data are in the [workflow-template](workflow-template) directory. To get a copy of that directory on to your system, copy the github web address of that directory, paste it into [GitZip here](http://kinolien.github.io/gitzip/), and then click download.

### 3. Modify the variables in the config.yaml file
Once you've downlonaded the workflow template, you can modify the variables in the [config.yaml](workflow-template/config.yaml) file as needed. For example, if you are processing a non-GLDS dataset, you will have to provide a text file containing a single-column list of unique sample identifiers. You will also need to indicate the paths to your input data (raw reads) and where to print your output data on your system and, if necessary, modify each variable to be consistent with the study you want to process. 

### 4. Run the workflow

To run the workflow, navigate to the directory holding the Snakefile, config.yaml, and other workflow files that you downloaded in step 2 then run the following command: 

```bash
snakemake --use-conda --conda-prefix ${CONDA_PREFIX}/envs -j 2 -p
```

**Parameter Definitions:**

* `--use-conda` – specifies to use the conda environments included in the workflow (these are specified in the [envs](envs) directory)
* `--conda-prefix` – indicates where the needed conda environments will be stored. Adding this option will also allow the same conda environments to be re-used when processing additional datasets, rather than makeing new environments each time you run the workflow. The value listed for this option, `${CONDA_PREFIX}/envs`, points to the default location for conda environments (note: the variable `${CONDA_PREFIX}` will be expanded to the appropriate location on whichever system it is run on).
* `-j` – assigns the number of jobs Snakemake should run concurrently (keep in mind that many of the thread and cpu parameters set in the config.yaml file will be multiplied by this)
* `-p` – specifies to print out each command being run to the screen

See `snakemake -h` and [Snakemake's documentation](https://snakemake.readthedocs.io/en/stable/) for more options and details.

---
