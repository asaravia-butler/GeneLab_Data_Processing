# GeneLab bioinformatics processing pipeline for one-channel microarray data

> **This page holds an overview and example code of how GeneLab processes one-channel Microarray datasets. Exact processing code and GL-DPPD-7100 revision used for specific datasets are available in the [GLDS_Processing_Scripts](https://developer.nasa.gov/asaravia/GeneLab_Data_Processing/tree/master/Microarray/1-channel_arrays/GLDS_Processing_Scripts) sub-directory and are also provided with their processed data in the [GeneLab Data Systems (GLDS) repository](https://genelab-data.ndc.nasa.gov/genelab/projects).**  

---

**Date:** Month day, 2020  
**Revision:** A   
**Document Number:** GL-DPPD-7100-A  

**Submitted by:**  
Homer Fogle (GeneLab Data Processing Team)

**Approved by:**  
Sylvain Costes (GeneLab Project Manager)  
Samrawit Gebre (GeneLab Deputy Project Manager and Interim GeneLab Configuration Manager)  
Amanda Saravia-Butler (GeneLab Data Processing Representative)  
Jonathan Galazka (GeneLab Project Scientist)

---

## Updates from previous revision

This version adds processing capability for additional Affymetrix ST type platforms as well as Nimblegen, Illumina Beadchip, and Genepix formatted Agilent expression arrays. Automated parsing of ISAtab metadata identifies paired sample data, technical replicates, timecourse factors, split-channel datafiles, and sample factor groups. Differential gene expression analysis for all pairwise group comparisons and gene level probe annotation have been added to the pipeline. 

---

# Table of contents  

- [**Software used**](#software-used)
- [**General processing overview with example code**](#general-processing-overview-with-example-code)
  - [**Input Data and Parameters**](#input-data-and-parameters)
  - [**Output Data**](#output-data)
  
---

# Software used  

### Required R Packages available from the CRAN repository:  

|Program   | Relevant Links|
|:---------|:--------------|
|shiny     | [https://cran.r-project.org/web/packages/shiny/index.html](https://cran.r-project.org/web/packages/shiny/index.html)|
|rmarkdown | [https://cran.r-project.org/web/packages/rmarkdown/index.html](https://cran.r-project.org/web/packages/rmarkdown/index.html)|
|knitr     | [https://cran.r-project.org/web/packages/knitr/index.html](https://cran.r-project.org/web/packages/knitr/index.html)|
|xfun      | [https://cran.r-project.org/web/packages/xfun/index.html](https://cran.r-project.org/web/packages/xfun/index.html)|
|DT        | [https://cran.r-project.org/web/packages/DT/index.html](https://cran.r-project.org/web/packages/DT/index.html)|
|R.utils   | [https://cran.r-project.org/web/packages/R.utils/index.html](https://cran.r-project.org/web/packages/R.utils/index.html)|
|dplyr     | [https://cran.r-project.org/web/packages/dplyr/index.html](https://cran.r-project.org/web/packages/dplyr/index.html)|
|tidyr     | [https://cran.r-project.org/web/packages/tidyr/index.html](https://cran.r-project.org/web/packages/tidyr/index.html)|
|statmod   | [https://cran.r-project.org/web/packages/statmod/index.html](https://cran.r-project.org/web/packages/statmod/index.html)|


### Required R Packages available from the Bioconductor repository:

|Program       | Relevant Links|
|:-------------|:--------------|
|Risa          | [https://www.bioconductor.org/packages/release/bioc/html/Risa.html](https://www.bioconductor.org/packages/release/bioc/html/Risa.html)|
|limma         | [https://bioconductor.org/packages/release/bioc/html/limma.html](https://bioconductor.org/packages/release/bioc/html/limma.html)|
|GEOquery      | [https://bioconductor.org/packages/release/bioc/html/GEOquery.html](https://bioconductor.org/packages/release/bioc/html/GEOquery.html)|
|ArrayExpress  | [https://www.bioconductor.org/packages/release/bioc/html/ArrayExpress.html](https://www.bioconductor.org/packages/release/bioc/html/ArrayExpress.html)|
|STRINGdb      | [https://www.bioconductor.org/packages/release/bioc/html/STRINGdb.html](https://www.bioconductor.org/packages/release/bioc/html/STRINGdb.html)|
|AnnotationDbi | [https://www.bioconductor.org/packages/release/bioc/html/AnnotationDbi.html](https://www.bioconductor.org/packages/release/bioc/html/AnnotationDbi.html)|
|oligo         | [https://www.bioconductor.org/packages/release/bioc/html/oligo.html](https://www.bioconductor.org/packages/release/bioc/html/oligo.html)|
|affy          | [http://bioconductor.org/packages/release/bioc/html/affy.html](http://bioconductor.org/packages/release/bioc/html/affy.html)|
|illuminaio    | [http://bioconductor.org/packages/release/bioc/html/illuminaio.html](http://bioconductor.org/packages/release/bioc/html/illuminaio.html)|
|HELP          | [https://www.bioconductor.org/packages/release/bioc/html/HELP.html](https://www.bioconductor.org/packages/release/bioc/html/HELP.html)|


> Exact versions are available along with the processing code for each specific dataset in the [GLDS_Processing_Scripts](https://developer.nasa.gov/asaravia/GeneLab_Data_Processing/tree/master/Microarray/1-channel_arrays/GLDS_Processing_Scripts) sub-directory. 

---

## 1. Raw Data Import

### For Affymetrix CEL Data
```R
raw <- oligo::read.celfiles(datapaths)
```
**Parameter Definitions:**
*	`raw <-` – specifies the variable that will store the results within in our R environment

*	`read.celfiles()` – the Oligo function we are calling, with the following parameters set within it

*	`datapaths` – the paths of input CEL files

### For NimbleGen XYS Data
```R
raw <- oligo::read.xysfiles(datapaths)
```
**Parameter Definitions:**
*	`raw <-` – specifies the variable that will store the results within in our R environment

*	`read.xysfiles()` – the Oligo function we are calling, with the following parameters set within it

*	`datapaths` – the paths of input XYS files

### For NimbleGen PAIR Data
```R
raw <- HELP::readPairs(datapaths)
```
**Parameter Definitions:**
*	`raw <-` – specifies the variable that will store the results within in our R environment

*	`readPairs()` – the HELP function we are calling, with the following parameters set within it

*	`datapaths` – the paths of input XYS files

### For Agilent RAW Data
```R
raw<-limma::read.maimages(filenames, source="agilent", path=dirname, green.only=TRUE, samplenames)
```
**Parameter Definitions:**
*	`raw <-` – specifies the variable that will store the results within in our R environment

*	`read.maimages()` – the Limma function we are calling, with the following parameters set within it

*	`filenames` – the names of input RAW files

* `source="agilent"` - selects the Agilent data format

* `path=dirname` - the directory path containing input files

* `green.only=TRUE` - specifies single channel data

* `samplenames` - sample names associated with each array

### For Agilent GPR Data
```R
raw<-limma::read.maimages(filenames, source="genepix", path=dirname, wt.fun=wtflags(weight=0,cutoff=-50), green.only=TRUE, names = samplenames)
```
**Parameter Definitions:**
*	`raw <-` – specifies the variable that will store the results within in our R environment

*	`read.maimages()` – the Limma function we are calling, with the following parameters set within it

*	`filenames` – the names of input RAW files

* `source="genepix"` - selects the GenePix data format

* `path=dirname` - the directory path containing input files

* `wt.fun=wtflags(weight=0,cutoff=-50)` - flags spots at cutoff intensity for normalization

* `green.only=TRUE` - specifies single channel data

* `samplenames` - sample names associated with each array

### For Illumina BeadChip TXT Data
```R
raw <- read.ilmn(filenames, ctrlfiles, path=dirname,ctrpath=dirname,probeid="Probe",annotation=c("TargetID", "SYMBOL"),expr="AVG_Signal",sep="\t", quote="\"")
```
**Parameter Definitions:**
*	`raw <-` – specifies the variable that will store the results within in our R environment

*	`read.ilmn()` – the Limma function we are calling, with the following parameters set within it

*	`filenames` – the names of the summary probe profile files

* `ctrlfiles` - the names of the summary control probe profile files

* `path=dirname` - the directory path containing input files

* `ctrpath=dirname` - the directory path containing input control probe files

* `probeid="Probe"` - the name of the probe identifier column

* `annotation=c("TargetID", "SYMBOL")` - the names of probe annotation columns

* `expr="AVG_Signal"` - the expression intensity column name

* `sep="\t"` - selects tab delimited format

* `quote="\""` - selects quote character

### For Illumina BeadChip IDAT Data
```R
raw <- read.idat(datapaths, bgxfile, annotation = c("RefSeq_ID","Symbol","Entrez_Gene_ID","Definition"),tolerance = 0L)
```
**Parameter Definitions:**
*	`raw <-` – specifies the variable that will store the results within in our R environment

*	`read.idat()` – the Limma function we are calling, with the following parameters set within it

*	`datapaths` – the paths of input IDAT files

* `bgxfile` - the bead manifest file (.bgx) to be read in

* `annotation = c("RefSeq_ID","Symbol","Entrez_Gene_ID","Definition")` - the names of probe annotation columns

* `tolerance = 0L` - the number of probe ID discrepancies allowed between the manifest and any of the IDAT files

## 2. Background Correction and Normalization

### For Affymetrix Expression Array Data
```R
data <- oligo::rma(raw, normalize = TRUE, background = TRUE)
```
**Parameter Definitions:**
*	`data <-` – specifies the variable that will store the results within in our R environment

*	`rma()` – the Oligo function we are calling, with the following parameters set within it

*	`raw` – raw data object generated in 1.

* `normalize = TRUE` - perform quantiles method normalization

* `background = TRUE` - perform RMA background correction

### For Affymetrix ST Array Data
```R
data <- oligo::rma(raw, target = "core", background=TRUE, normalize=TRUE)
```
**Parameter Definitions:**
*	`data <-` – specifies the variable that will store the results within in our R environment

*	`rma()` – the Oligo function we are calling, with the following parameters set within it

*	`raw` – raw data object generated in 1.

* `target = "core"` - perform gene level summarization

* `normalize = TRUE` - perform quantiles method normalization

* `background = TRUE` - perform RMA background correction

### For Agilent Array Data
```R
data <- backgroundCorrect(raw, method="normexp", offset=50)
data <- normalizeBetweenArrays(data, method="quantile")
```
**Parameter Definitions:**
*	`data <-` – specifies the variable that will store the results within in our R environment

*	`backgroundCorrect()` – the Limma function for background correction

*	`raw` – raw data object generated in 1.

* `method="normexp"` - perform normexp background correction

* `offset=50` - set offset value for background correction

*	`normalizeBetweenArrays()` – the Limma function for cross array normalization

* `method="quantile"` - perform quantile normalization

## 3. Data Filtering
```R
data.filt <- genefilter::nsFilter(data, require.entrez=TRUE,
    remove.dupEntrez=TRUE, var.func=IQR,
    var.cutoff=0.5, var.filter=TRUE,
    filterByQuantile=TRUE, feature.exclude="^AFFX")
```
**Parameter Definitions:**
*	`data.filt <-` – specifies the variable that will store the results within in our R environment

*	`nsFilter()` – the genefilter function we are calling, with the following parameters set within it

*	`data` – normalized data object generated in 2.

* `require.entrez=TRUE` - filter unannotated probes

* `remove.dupEntrez=TRUE` - filter duplicate gene level values

*	`var.func=IQR` – select probes by maximum interquartile range

* `var.cutoff=0.5` - set variance filter cutoff

*	`var.filter=TRUE` – filter by variance function

* `filterByQuantile=TRUE` - filter by quantiles

* `feature.exclude="^AFFX"` - filter control probes
