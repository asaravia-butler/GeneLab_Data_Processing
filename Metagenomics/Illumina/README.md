
# GeneLab bioinformatics processing protocol for Illumina metagenomics data

> **The document [`GL-DPPD-7107.md`](GL-DPPD-7107.md) holds an overview and some example commands of how GeneLab processes Illumina metagenomics datasets. Exact processing commands for specific datasets that have been released is available in the [GLDS_Processing_Scripts](GLDS_Processing_Scripts) sub-directory and is also provided with their processed data in the [GeneLab Data Systems (GLDS) repository](https://genelab-data.ndc.nasa.gov/genelab/projects).**  

**Developed and maintained by:**  
Michael D. Lee (Mike.Lee@nasa.gov)

---

<p align="center">
<a href="../images/GL-Illumina-metagenomics-overview.pdf"><img src="../images/GL-Illumina-metagenomics-overview.png"></a>
</p>

--- 

# Repository Links

* [Current processing protocol (GL-DPPD-7107.md)](GL-DPPD-7107.md)  
* [Processed dataset files](GLDS_Processing_Scripts)  
* [Template worflow files](workflow-template)  

---

# General workflow info
The processing is implemented as a [Snakemake](https://snakemake.readthedocs.io/en/stable/) workflow and utilizes [conda](https://docs.conda.io/en/latest/) environments. The workflow can be used even if unfamiliar with Snakemake and conda, but if wanting to learn more about those, [this Snakemake tutorial](https://snakemake.readthedocs.io/en/stable/tutorial/tutorial.html) within [Snakemake's documentation](https://snakemake.readthedocs.io/en/stable/) is a good place to start for that, and an introduction to conda with installation help and links to other resources can be found [here at Happy Belly Bioinformatics](https://astrobiomike.github.io/unix/conda-intro).  

> **Note on reference databases**  
> Many reference databases are relied upon throughout this workflow. They will be installed and setup automatically the first time the workflow is run. All together, after installed and unpacked, they will take up about 240 GB of storage, but they may also require up to 500GB during installation and initial un-packing, so be sure there is enough room on your system.

## Utilizing the workflow

* [Installing conda and snakemake](#installing-conda-and-snakemake)  
* [Downloading the workflow template files](#downloading-the-workflow-template-files)  
* [Modifying the variables in the config.yaml file](#modifying-the-variables-in-the-configyaml-file)  
* [Running the workflow](#running-the-workflow)  

### Installing conda and snakemake
We recommend installing a Miniconda, Python3 version appropriate for your system, as exemplified in [the above link](https://astrobiomike.github.io/unix/conda-intro#getting-and-installing-conda).  

Once we have conda, we can install the latest snakemake like so:

```bash
conda install -c conda-forge -c bioconda -c defaults snakemake
```

### Downloading the workflow template files
All files required for utilizing the workflow are in the [workflow-template](workflow-template) directory. To get a copy of that directory on to our system, we can copy the address of that directory, paste it into [GitZip here](http://kinolien.github.io/gitzip/), and then click download:

<p align="center">
<a href="../images/gitzip-ex.png"><img src="../images/gitzip-ex.png"></a>
</p>

### Modifying the variables in the config.yaml file
We then need to modify the variables in our downloaded version of the [config.yaml](workflow-template/config.yaml) file as needed in order to match our dataset and system setup. These include things like specifying a file with our unique sample identifiers, pointing to where our starting reads are located, and pointing to where our reference databases are to be stored (these should be specified as relative or full paths – if unfamiliar with these, one place you can learn more is [here](https://astrobiomike.github.io/unix/getting-started#the-unix-file-system-structure).  

For example, if we had paired-read data for 2 samples located in `../Raw_Data/` relative to our workflow directory, that look like this:

```bash
ls ../Raw_Data/
```

```
Sample-1_R1_raw.fastq.gz
Sample-1_R2_raw.fastq.gz
Sample-2_R1_raw.fastq.gz
Sample-2_R2_raw.fastq.gz
```

And we had the unique sample identifiers in a file called `unique-sample-IDs.txt`:

```bash
cat unique-sample-IDs.txt
```

```
Sample-1
Sample-2
```

We'd want to set up the first few variables in our config.yaml file such that they hold that information, including the particular suffix our starting reads have, e.g.:

<p align="center">
<a href="../images/config-ex.png"><img src="../images/config-ex.png"></a>
</p>

The last variable we *need* to specify for our system is the `REF_DB_ROOT_DIR`, which is where we want the reference databases to be stored. As noted above, they will take up about 240 GB of storage after setup and fully unpacked, but they may also require up to 500GB during installation and initial un-packing, so we need to be sure there is enough room on our system before trying to run the workflow.

The rest of the variables in the config.yaml file only need to be changed if wanted. 

### Running the workflow

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
