# Workflow information and usage instructions


## General workflow info
The processing is implemented as a [Snakemake](https://snakemake.readthedocs.io/en/stable/) workflow and utilizes [conda](https://docs.conda.io/en/latest/) environments. The workflow can be used even if unfamiliar with Snakemake and conda, but if wanting to learn more about those, [this Snakemake tutorial](https://snakemake.readthedocs.io/en/stable/tutorial/tutorial.html) within [Snakemake's documentation](https://snakemake.readthedocs.io/en/stable/) is a good place to start for that, and an introduction to conda with installation help and links to other resources can be found [here at Happy Belly Bioinformatics](https://astrobiomike.github.io/unix/conda-intro).  

## Reference database info
The database we use was built with kraken2 v2.1.1 as detailed below, and by default will be downloaded to run with the above workflow (it's ~4.3 GB uncompressed). 

---

### Kraken2 human database build

> The following was performed on 29-Nov-2020 with kraken v2.1.1.

**Downloading human reference (takes ~2 minutes as run here):**

```bash
kraken2-build --download-library human --db kraken2-human-db --threads 30 --no-masking
```

**Downloading NCBI taxonomy info needed (takes ~10 minutes):**

```bash
kraken2-build --download-taxonomy --db kraken2-human-db/
```

**Building database (takes ~20 minutes as run here):**

```bash
kraken2-build --build --db kraken2-human-db/ --threads 30
```

**Removing intermediate files:**

```bash
kraken2-build --clean --db kraken2-human-db/
```

---

### Download database as built on 29-Nov-2020
The reference database 3GB compressed and ~4.3GB uncompressed. If using the accompanying Snakemake workflow, it will download and use this reference database. It can also be downloaded and unpacked by itself with the following:

```bash
curl -L -o kraken2-human-db.tar.gz https://ndownloader.figshare.com/files/25627058

tar -xzvf kraken2-human-db.tar.gz
```

---

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

A quick example (once the reference database has been created or downloaded from above) can be run with the files included in the [workflow-template](workflow-template) directory after specifying a location for the reference database in the [config.yaml](workflow-template/config.yaml) file.

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
