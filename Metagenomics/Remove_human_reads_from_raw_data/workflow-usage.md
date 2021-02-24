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
All files required for utilizing the GeneLab workflow for removing human reads from metagenomics data are in the [workflow-template](workflow-template) directory. To get a copy of that directory on to your system, copy the github web address of that directory, paste it into [GitZip here](http://kinolien.github.io/gitzip/), and then click download.

### 3. Modify the variables in the config.yaml file
Once you've downlonaded the workflow template, you can modify the variables in the [config.yaml](workflow-template/config.yaml) file as needed. For example, if you are processing a non-GLDS dataset, you will have to provide a text file containing a single-column list of unique sample identifiers (if you are running the example dataset, this file is provided in the [workflow-template](workflow-template) directory [here](workflow-template/unique-sample-names.txt). You will also need to indicate the paths to your input data (raw reads and the human kraken2 database if it has already been downloaded - if it has not been downloaded, indicate the path to where you want it downloaded) and where to print your output data on your system. Additionally, if necessary, you'll need to modify each variable in the [config.yaml](workflow-template/config.yaml) file to be consistent with the study you want to process and the machine you're using. 

A quick example (once the reference database has been created or downloaded from above) can be run with the files included in the [workflow-template](workflow-template) directory after specifying a location for the reference database in the [config.yaml](workflow-template/config.yaml) file.

### 4. Run the workflow

To run the workflow, navigate to the directory holding the Snakefile, config.yaml, and other workflow files that you downloaded in step 2 then run the following command: 

```bash
snakemake --use-conda --conda-prefix ${CONDA_PREFIX}/envs -j 2 -p
```

* `--use-conda` – specifies to use the conda environments included in the workflow (these are specified in the [envs](envs) directory)
* `--conda-prefix` – indicates where the needed conda environments will be stored. Adding this option will also allow the same conda environments to be re-used when processing additional datasets, rather than makeing new environments each time you run the workflow. The value listed for this option, `${CONDA_PREFIX}/envs`, points to the default location for conda environments (note: the variable `${CONDA_PREFIX}` will be expanded to the appropriate location on whichever system it is run on).
* `-j` – assigns the number of jobs Snakemake should run concurrently (keep in mind that many of the thread and cpu parameters set in the config.yaml file will be multiplied by this)
* `-p` – specifies to print out each command being run to the screen

See `snakemake -h` and [Snakemake's documentation](https://snakemake.readthedocs.io/en/stable/) for more options and details.

---

## Reference database info
The database we use was built with kraken2 v2.1.1 as detailed below, and by default will be downloaded to run with this workflow (it's ~4.3 GB uncompressed). 

---

### Kraken2 human database build

> The following was performed on 29-Nov-2020 with kraken v2.1.1.

**Download human reference (takes ~2 minutes as run here):**

```bash
kraken2-build --download-library human --db kraken2-human-db --threads 30 --no-masking
```

**Download NCBI taxonomy info needed (takes ~10 minutes):**

```bash
kraken2-build --download-taxonomy --db kraken2-human-db/
```

**Build the database (takes ~20 minutes as run here):**

```bash
kraken2-build --build --db kraken2-human-db/ --threads 30
```

**Remove intermediate files:**

```bash
kraken2-build --clean --db kraken2-human-db/
```

---

### Download database as built on 29-Nov-2020
The reference database is 3GB compressed and ~4.3GB uncompressed. If using the accompanying Snakemake workflow, it will download and use this reference database. It can also be downloaded and unpacked independently by running the following commands:

```bash
curl -L -o kraken2-human-db.tar.gz https://ndownloader.figshare.com/files/25627058

tar -xzvf kraken2-human-db.tar.gz
```

---
