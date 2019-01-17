<p align="left">
  <img src="./Dumpling.png">
</p>

# Overview

Welcome to the GitHub repository for DiMSum: A pipeline for pre-processing of paired-end reads from deep mutational scannning (DMS) data.

# Required Software

To run the DiMSum pipeline you will need the following software and associated packages:

* **[R](https://www.r-project.org/) >=v3.5.2** (data.table, ggplot2, GGally, hexbin, optparse, parallel, plyr, reshape2, seqinr, ShortRead)
* **[FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) v0.11.3**
* **[cutadapt](https://cutadapt.readthedocs.io/en/stable/) v1.16**
* **[USEARCH 32-bit](https://drive5.com/usearch/download.html) v10.0**
* **fastx_collapser** from the [FASTX Toolkit](http://hannonlab.cshl.edu/fastx_toolkit/download.html) v0.0.13

# Installation and loading

Open R and enter:

```
# Install
if(!require(devtools)) install.packages("devtools")
devtools::install_github("lehner-lab/DiMSum")

# Load
library(DiMSum)

# Help
?dimsum
```

# DiMSum command-line tool

Clone the DiMSum repository and install the R package locally. The * must be replaced by what is actually downloaded and built.

```
git clone https://github.com/lehner-lab/DiMSum.git
R CMD build DiMSum
R CMD INSTALL DiMSum_*.tar.gz
```
Add the cloned DiMSum repository base directory to your path. You can do this by adding the following line at the bottom of your ~/.bashrc file:
```
export PATH=CLONED_DIMSUM_REPOSITORY:$PATH
```
Get a description of DiMSum command-line arguments with the following:
```
DiMSum -h
```

# Pipeline

The DiMSum pipeline processes paired-end reads (in FASTQ format) from deep mutational scanning (DMS) experiments to produce variant counts for each sample. These counts are suitable for use in downstream analyses of epistasis and [protein structure determination](https://github.com/lehner-lab/DMS2structure).

## Stage 1: DEMULTIPLEX READS

Demultiplex samples and trim read barcodes using cutadapt (optional). This stage is run if a barcode design file is supplied (see 'barcodeDesignPath' argument). Stage-specific arguments: 'barcodeDesignPath' and 'barcodeErrorRate'.

## Stage 2: ASSESS READ QUALITY

Produce raw read quality reports using FastQC.

## Stage 3: TRIM CONSTANT REGIONS

Remove constant region sequences from read 5’ and 3’ ends using cutadapt. 5' constant region sequences are required (3' sequences are optional). By default the sequences of 3' constant regions are assumed to be the reverse complement of 5' constant region sequences. Stage-specific arguments: 'cutadaptCut5First', 'cutadaptCut5Second', 'cutadaptCut3First', 'cutadaptCut3Second', 'cutadapt5First', 'cutadapt5Second', 'cutadapt3First', 'cutadapt3Second', 'cutadaptMinLength', 'cutadaptErrorRate'.

## Stage 4: ALIGN PAIRED-END READS

Align overlapping paired-end reads using USEARCH (cis libraries only i.e. 'transLibrary'=F). Stage-specific arguments: 'usearchMinQual', 'usearchMaxee', 'usearchMinlen', 'usearchMinovlen'.

## Stage 5: COUNT UNIQUE VARIANTS

Tally counts of unique variant sequences using FASTX-Toolkit.

## Stage 6: MERGE SAMPLE STATISTICS

Combine sample-wise variant counts and statistics to produce a unified results data.table. Variant counts are aggregated across technical replicates.

# Experimental design file

To run this pipeline, you will first need to describe your experimental design (e.g. in MSExcel) and save this as a tab-separated plain text file. You can download [this](./example_experimentDesign.txt) file to use as a template.

Your file must have the following columns:
* **sample_name** A sensible sample name e.g. 'input1' (only alphanumeric characters allowed)
* **experiment** An integer identifier for each unique experiment i.e. a set of input and output replicates relating to the same original plasmid library transformation or input biological replicate (a strictly positive integer)
* **selection_id** An integer inidicating whether samples were sequenced before (0) or after (1) selection. Subsequent (serial) rounds of selection are indicated by higher numbers i.e. 2, 3, etc. (a positive integer, zero included)
* **biological_replicate** An integer indicating the output biological replicate (a strictly positive integer). This field must be blank (empty string) for input samples (each input sample corresponds to a separate biological replicate i.e. experiment).
* **technical_replicate** An integer indicating the technical replicate (a strictly positive integer). Leave this column blank (empty string) when no technical replicates are present.
* **pair1** File name of the first read in a given pair.
* **pair2** File name of the second read in a given pair.


# Barcode design file

If your raw FASTQ sequencing files contain multiplexed samples you will need to provide a tab-separated plain text file describing how barcodes map to samples. You can download [this](./example_barcodeDesign.txt) file to use as a template.



(Vector illustration credit: <a href="https://www.vecteezy.com">Vecteezy!</a>)
