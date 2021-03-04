# GeneLab bioinformatics processing pipeline for two-channel microarray data

> **This page holds an overview and example code of how GeneLab processes two-channel Microarray datasets. Exact processing code and GL-DPPD-710X revision used for specific datasets are available in the [GLDS_Processing_Scripts](https://developer.nasa.gov/asaravia/GeneLab_Data_Processing/tree/master/Microarray/2-channel_arrays/GLDS_Processing_Scripts) sub-directory and are also provided with their processed data in the [GeneLab Data Systems (GLDS) repository](https://genelab-data.ndc.nasa.gov/genelab/projects).**  

---

**Date:** Month day, 2020  
**Revision:** -   
**Document Number:** GL-DPPD-710X  

**Submitted by:**  
Homer Fogle (GeneLab Data Processing Team)

**Approved by:**  
Sylvain Costes (GeneLab Project Manager)  
Samrawit Gebre (GeneLab Deputy Project Manager and Interim GeneLab Configuration Manager)  
Amanda Saravia-Butler (GeneLab Data Processing Representative)  
Jonathan Galazka (GeneLab Project Scientist)

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


> Exact versions are available along with the processing code for each specific dataset in the [GLDS_Processing_Scripts](https://developer.nasa.gov/asaravia/GeneLab_Data_Processing/tree/master/Microarray/2-channel_arrays/GLDS_Processing_Scripts) sub-directory. 

---

## 1. Raw Data Import

### For Agilent GPR Data
```R
RG <- read.maimages(files_in_assay, source="genepix", path=path,wt.fun=wtflags(weight=0,cutoff=-50))
```
**Parameter Definitions:**
*	`RG <-` – specifies the variable that will store the results within in our R environment

*	`read.maimages()` – the Limma function we are calling, with the following parameters set within it

*	`files_in_assay` – the filenames of input GPR files

* `source="genepix"` - selects Genepix formatted array data

* `path=path` - the parent directory path of input files

* `wt.fun=wtflags(weight=0,cutoff=-50)` - function to calculate spot quality weights

### For Agilent RAW Data
```R
RG <- read.maimages(files_in_assay, source="agilent", path=path)
```
**Parameter Definitions:**
*	`RG <-` – specifies the variable that will store the results within in our R environment

*	`read.maimages()` – the Limma function we are calling, with the following parameters set within it

*	`files_in_assay` – the filenames of input RAW files

* `source="genepix"` - selects Agilent formatted array data

* `path=path` - the parent directory path of input files

### For SPOT Data
```R
RG <- read.maimages(files_in_assay, source="spot", path=path)
```
**Parameter Definitions:**
*	`RG <-` – specifies the variable that will store the results within in our R environment

*	`read.maimages()` – the Limma function we are calling, with the following parameters set within it

*	`files_in_assay` – the filenames of input GPR files

* `source="spot"` - selects spot formatted array data

* `path=path` - the parent directory path of input files

### For Imagene Data
```R
RG <- read.maimages(files_in_assay, source="imagene", path=path)
```
**Parameter Definitions:**
*	`RG <-` – specifies the variable that will store the results within in our R environment

*	`read.maimages()` – the Limma function we are calling, with the following parameters set within it

*	`files_in_assay` – the filenames of input imagene files

* `source="imagene"` - selects imagene formatted array data

* `path=path` - the parent directory path of input files

### For Generic Data
```R
RG <- read.maimages(files_in_assay, source="generic", path=path)
```
**Parameter Definitions:**
*	`RG <-` – specifies the variable that will store the results within in our R environment

*	`read.maimages()` – the Limma function we are calling, with the following parameters set within it

*	`files_in_assay` – the filenames of input generic tabular files

* `source="generic"` - selects generic formatted array data

* `path=path` - the parent directory path of input files

### For Genepix Data
```R
RG <- read.maimages(files_in_assay,source="genepix",path = path,wt.fun=wtflags(weight=0,cutoff=-50))
```
**Parameter Definitions:**
*	`RG <-` – specifies the variable that will store the results within in our R environment

*	`read.maimages()` – the Limma function we are calling, with the following parameters set within it

*	`files_in_assay` – the filenames of input GPR files

* `source="genepix"` - selects genepix formatted array data

* `path=path` - the parent directory path of input files

* `wt.fun=wtflags(weight=0,cutoff=-50)` - function to calculate spot quality weights

### For Nimblegen Data
```R
RG <-  Ringo::readNimblegen(hybesFile = "nimblegen_targets.txt",spotTypesFile = "spottypes.txt",path = path, headerPattern = "# software=DEVA", verbose = TRUE)
```
**Parameter Definitions:**
*	`RG <-` – specifies the variable that will store the results within in our R environment

*	`Ringo::readNimblegen()` – the Ringo function we are calling, with the following parameters set within it

*	`hybesFile = "nimblegen_targets.txt"` – metadata file in Limma targets format

* `spotTypesFile = "spottypes.txt"` - metadata file in Limma spottypes format

* `path=path` - the parent directory path of input files

* `headerPattern = "# software=DEVA"` - pattern used to identify explantory header lines in the supplied pair-format files

* `verbose = TRUE` - progress output to STDOUT
