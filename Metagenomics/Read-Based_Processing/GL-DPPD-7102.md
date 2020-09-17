# Bioinformatics pipeline for Illumina metagenomics data (read-based analysis)  

> **This page holds an overview and some example code of how GeneLab processes Illumina metagenomics datasets using a read-based analysis approach. Exact processing code for specific datasets that have been released are available in the [GLDS_Processing_Scripts](GLDS_Processing_Scripts) sub-directory and are also provided with their processed data in the [GeneLab Data Systems (GLDS) repository](https://genelab-data.ndc.nasa.gov/genelab/projects).**  

---

**Date:** April 06, 2020  
**Revision:** -  
**Document Number:** GL-DPPD-7102  

**Submitted by:**  
Michael D. Lee (GeneLab Data Processing Team)

**Approved by:**  
Sylvain Costes (GeneLab Project Manager)  
Samrawit Gebre (GeneLab Deputy Project Manager)  
Homer Fogle (GeneLab Data Processing Representative)  
Jonathan Galazka (GeneLab Project Scientist)  
Anushree Sonic (Genelab Configuration Manager)  

---

# Table of contents  

- [Software used](#software-used)
- [General processing overview with example code](#general-processing-overview-with-example-code)
  - [1. Raw Data QC](#1-raw-data-qc)
    - [Compile Raw Data QC](#compile-raw-data-qc)
  - [2. Quality filtering](#2-quality-filtering)
  - [3. Filtered Data QC](#3-filtered-data-qc)
    - [Compile Filtered Data QC](#compile-filtered-data-qc)
  - [4. Taxonomic profiling](#4-taxonomic-profiling)
    - [Assigning initial taxonomy](#assigning-initial-taxonomy)
    - [Refining taxonomy and getting relative abundances](#refining-taxonomy-and-getting-relative-abundances)
    - [Combining multiple samples into one table](#combining-multiple-samples-into-one-table)
    - [Getting full lineage of each taxonomy and adding to final taxonomy table](#getting-full-lineage-of-each-taxonomy-and-adding-to-final-taxonomy-table)
  - [5. Functional profiling](#5-functional-profiling)
    - [Functional profiling of samples](#functional-profiling-of-samples)
    - [Merging multiple sample functional profiles into one table](#merging-multiple-sample-functional-profiles-into-one-table)
    - [Generating unstratified versions of the tables](#generating-unstratified-versions-of-the-tables)
    - [Normalizing gene families and pathway abundance tables](#normalizing-gene-families-and-pathway-abundance-tables)
    - [Generating a normalized gene family table that is summarized by Kegg Orthologs](#generating-a-normalized-gene-family-table-that-is-summarized-by-kegg-orthologs)

---

# Software used

|Program|Version*|Relevant Links|
|:------|:------:|:-------------|
|FastQC|`fastqc -v`|[https://www.bioinformatics.babraham.ac.uk/projects/fastqc/](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)|
|MultiQC|`multiqc -v`|[https://multiqc.info/](https://multiqc.info/)|
|bbduk|`bbduk.sh --version`|[https://jgi.doe.gov/data-and-tools/bbtools/bb-tools-user-guide/bbduk-guide/](https://jgi.doe.gov/data-and-tools/bbtools/bb-tools-user-guide/bbduk-guide/)|
|Kraken2|`kraken2 -v`|[https://ccb.jhu.edu/software/kraken2/index.shtml](https://ccb.jhu.edu/software/kraken2/index.shtml)|
|Bracken|`conda list \| grep bracken`|[https://ccb.jhu.edu/software/bracken/index.shtml](https://ccb.jhu.edu/software/bracken/index.shtml)|
|TaxonKit|`taxonkit -h`|[https://github.com/shenwei356/taxonkit](https://github.com/shenwei356/taxonkit)|
|HUMAnN2|`humann2 --version`|[https://huttenhower.sph.harvard.edu/humann](https://huttenhower.sph.harvard.edu/humann)|

>**\*** Exact versions are available along with the processing code for each specific dataset.

---

# General processing overview with example code  

> Exact processing code for specific datasets are available in the [GLDS_Processing_Scripts](GLDS_Processing_Scripts) sub-directory of this repository, as well as being provided with their processed data in the [GeneLab Data Systems (GLDS) repository](https://genelab-data.ndc.nasa.gov/genelab/projects).  

---

## 1. Raw Data QC  

```
fastqc -o raw_fastqc_output *.fastq.gz
```

**Parameter Definitions:**

* `-o` – the output directory to store results
* `*.fastq.gz` – the input reads are specified as a positional argument, and can be given all at once with wildcards like this, or as individual arguments with spaces in between them

**Input Data:**

* fastq, compressed or uncompressed

**Output Data:**

* fastqc.html (FastQC output html summary)
* fastqc.zip (FastQC output data)


<br>  

### Compile Raw Data QC  

```
multiqc -o raw_multiqc_output raw_fastqc_output
```

**Parameter Definitions:**

*	`-o` – the output directory to store results
*	`raw_fastqc_output/` – the directory holding the output data from the fastqc run, provided as a positional argument

**Input Data:**

* fastqc.zip (FastQC output data)

**Output Data:**

* multiqc_report.html (multiqc output html summary)
* multiqc_data (directory containing multiqc output data)

<br>  

---

## 2. Quality filtering

```
bbduk.sh in=raw-R1.fastq.gz in2=raw-R2.fastq.gz out1=R1-trimmed.fastq.gz out2=R2-trimmed.fastq.gz \
         ref=ref-adapters.fa ktrim=l k=17 ftm=5 qtrim=rl trimq=10 mlf=0.5 maxns=0 > bbduk.log 2>&1
```

**Parameter Definitions:**

*	`in` and `in2` – specifies the forward and reverse input reads, respectively

*	`out1` and `out2` – specifies the forward and reverse output reads, respectively

*	`ref` – specifies a fasta file holding potential adapter sequences (comes with bbduk installation)

*	`ktrim` – specifies to trim adapters from the 5’ end (left) if found

*	`k` – sets minimum length of kmer match to identify adapter sequences (provided by the “ref” file above)

*	`ftm` – sets a multiple of expected length the sequence should be (handles poor additional bases that are sometimes present, see “Force-Trim Modulo” section on this page)

*	`qtrim` – sets quality-score-based trimming to be applied to left and right sides

*	`trimq` – sets the score to use for PHRED-algorithm trimming

*	`mlf` – sets the minimum length of reads retained based on their initial length

*	`maxns` – sets the maximum number of Ns allowed in a read before it will be filtered out

*	`> bbduk.log 2>&1` – redirects the stderr and stdout (which is informative in this case) to a log file for saving

**Input Data:**

* fastq, compressed or uncompressed (original reads)

**Output Data:**

* fastq, compressed or uncompressed (filtered reads)
* tsv (per sample read counts before and after filtering)
* txt (log file of standard output and error from bbduk run)

<br>

---

## 3. Filtered Data QC
```
fastqc -o filtered_fastqc_output/ filtered*.fastq.gz
```

**Parameter Definitions:**

*	`-o` – the output directory to store results  
*	`filtered*.fastq.gz` – the input reads are specified as a positional argument, and can be given all at once with wildcards like this, or as individual arguments with spaces in between them  

**Input Data:**

* fastq, compressed or uncompressed (filtered reads)

**Output Data:**

* fastqc.html (FastQC output html summary)
* fastqc.zip (FastQC output data)

<br>

### Compile Filtered Data QC
```
multiqc -o filtered_multiqc_output  filtered_fastqc_output
```

**Parameter Definitions:**

*	`-o` – the output directory to store results
*	`filtered_fastqc_output` – the directory holding the output data from the fastqc run, provided as a positional argument

**Input Data:**

* fastqc.zip (FastQC output data)

**Output Data:**

* multiqc_report.html (multiqc output html summary)
* multiqc_data (directory containing multiqc output data)

<br>

---

## 4. Taxonomic profiling 
The following uses the kraken2 ["standard" database](https://ccb.jhu.edu/software/kraken2/index.shtml?t=manual#standard-kraken-2-database) as built on 28-Jan-2019.  

<br>

### Assigning initial taxonomy
```
kraken2 –-db ~/kraken2-standard/ --threads 40 --report kraken2-report-sample-1.tsv --gzip-compressed \
        --paired --output kraken2-sample-1.out R1-paired.fastq.gz R2-paired.fastq.gz
```

**Parameter Definitions:**  

*	`--db` – specifies the path to (location of) the kraken2 database directory

*	`--threads 40` – specifies the number of threads to run in parallel

*	`--report` – specifies the name of the output report file

*	`--gzip-compressed` – tells the program the input files are gzip-compressed

*	`--paired` – tells the program the input files are paired reads (not needed if single-end reads)

*	`--output` – specifies the name of the general output file

*	The last two positional arguments are the forward and reverse input reads (forward only for single-end)

<br>

### Refining taxonomy and getting relative abundances
```
bracken -d ~/kraken2-standard/ -i kraken2-report-sample-1.tsv -o bracken-taxonomy-sample-1.tsv
```

**Parameter Definitions:**  

*	`-d` – specifies the path to (location of) the kraken2 database directory

*	`-i` – specifies the input report file that was output from the above kraken2 command

*	`-o` – specifies the bracken output file

<br>

### Combining multiple samples into one table
```
combine_bracken_outputs.py --files bracken-taxonomy-sample-1.tsv bracken-taxonomy-sample-2.tsv \
                           --names Sample1,Sample2 -o combined-taxonomy.tmp
```

**Parameter Definitions:** 

*	`--files` – specifies the input files, output from the bracken command above

*	`--names` – specifies the names of the samples to put in the column headers (must be in same order as sample files provided)

*	`-o` – specifies the name of the output, combined file

<br>

### Getting full lineage of each taxonomy and adding to final taxonomy table
```
tail -n +2 combined-taxonomy.tmp | cut -f 2 \
taxonkit lineage | taxonkit reformat -r NA | \
cut -f 3 | tr “;” “\t” | cut -f 1-7 > lineages.tmp

cat <(printf “domain\tphylum\tclass\torder\tfamily\tgenus\tspecies\n”) lineages.tmp > lineages-tab.tmp

paste lineages-tab.tmp <(cut -f 2- combined-taxonomy.tmp) > taxonomy.tsv
rm *.tmp
```

**Input Data:**

* fastq, compressed or uncompressed (filtered reads)

**Output Data:**

* taxonomy.tsv (read counts and relative abundances of classified taxa)

<br>

---

## 5. Functional profiling 
HUMAnN2 treats forward and reverse reads independently, and does not take them as input separately, unfortunately. So, for paired-end studies, the reads need to be concatenated first such that each sample has one fastq.gz that holds both forward and reverse reads. 

<br>

### Functional profiling of samples
```
mkdir genefamily-results/ pathabundance-results/ pathcoverage-results/

humann2 --input R1-and-R2-trimmed-reads.fastq.gz --output sample_functional_output_dir/ \
        --threads 20 --remove-temp-output

cp sample_functional_output_dir/*genefamilies.tsv genefamily-results/
cp sample_functional_output_dir/*abundance.tsv pathabundance-results/
cp sample_functional_output_dir/*coverage.tsv pathcoverage-results/
```

**Parameter Definitions:**  

*	`--input` – the input sample reads

*	`--output` – the directory to hold the output files

*	`--threads` – specifies the number of threads to run in parallel

*	`--remove-temp-output` – delete the temp files after finishing

<br>

### Merging multiple sample functional profiles into one table
```
humann2_join_tables -i genefamily-results/ -o genefamilies.tsv

humann2_join_tables -i pathabundance-results/ -o pathabundances.tsv

humann2_join_tables -i pathcoverage-results/ -o pathcoverages.tsv
```

**Parameter Definitions:**  

*	`-i` – the directory holding the input tables

*	`-o` – the name of the output combined table

<br>

### Generating unstratified versions of the tables
```
humann2_split_stratified_table -i genefamilies.tsv -o ./
rm genefamilies.tsv

humann2_split_stratified_table -i pathabundances.tsv -o ./
rm pathabundances.tsv

humann2_split_stratified_table -i pathcoverages.tsv -o ./
rm pathcoverages.tsv
```

**Parameter Definitions:**  

*	`-i` – the input combined table

*	`-o` – output directory (here specifying current directory)

<br>

### Normalizing gene families and pathway abundance tables
```
humann2_renorm_table -i genefamilies_unstratified.tsv -o genefamilies-cpm.tsv --update-snames

humann2_renorm_table -i pathabundances_unstratified.tsv -o pathabundances-cpm.tsv --update-snames
```

**Parameter Definitions:**  

*	`-i` – the input combined table

*	`-o` – name of the output normalized table

*	`--update-snames` – change suffix of column names to “-CPM”

<br>

### Generating a normalized gene family table that is summarized by Kegg Orthologs
```
humann2_regroup_table -i genefamilies-cpm.tsv -g uniref90_ko | humann2_rename_table \
                      -o genefaimlies-KO-cpm.tsv -n kegg-orthology
```

**Parameter Definitions:**  

*	`-i` – the input table

*	`-g` – the map to use to group uniref IDs into Kegg orthologs

*	`|` – sending that output into the next humann2 command to add human-readable Kegg orthology names

*	`-o` – the final output file name

*	`-n` – specifying we are converting Kegg orthology IDs into Kegg orthology human-readable names

**Input Data:**

* fastq, compressed or uncompressed (filtered reads, forward and reverse reads concatenated if paired-end)

**Output Data:**

* genefamilies_unstratified.tsv (gene-family abundances)
* genefamilies_stratified.tsv (grouped by limited taxonomy)
* genefamilies-cpm.tsv (normalized)
* genefamilies-KO-cpm.tsv (normalized, grouped by Kegg Orthologs)
* pathabundances_unstratified.tsv (pathway abundances)
* pathabundances_stratified.tsv (grouped by limited taxonomy)
* pathabundances-cpm.tsv (normalized)
* pathcoverages_unstratified.tsv (pathway coverages)
* pathcoverages_stratified.tsv (grouped by limited taxonomy)
