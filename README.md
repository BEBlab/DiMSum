<p align="left">
  <img src="./Dumpling.png" width="100">
</p>

Welcome to the GitHub repository for DiMSum: A pipeline for analyzing deep mutational scanning (DMS) data and diagnosing common experimental pathologies.


# Table Of Contents

* **1. [Installation Instructions](docs/INSTALLATION.md)**
* **2. [Pipeline Overview]()**
* **3. [Input File Formats]()**

# Pipeline Overview

The DiMSum pipeline processes raw sequencing reads (in FASTQ format) from deep mutational scanning (DMS) experiments to produce variant counts, fitness and error estimates. These estimates are suitable for use in downstream analyses of epistasis and [protein structure determination](https://github.com/lehner-lab/DMS2structure).

## Stage 0: DEMULTIPLEX READS

Demultiplex samples and trim read barcodes using cutadapt (optional). This stage is run if a barcode design file is supplied (see 'barcodeDesignPath' argument). Stage-specific arguments: 'barcodeDesignPath' and 'barcodeErrorRate'.

## Stage 1: ASSESS READ QUALITY

Produce raw read quality reports using FastQC (and unzip and split FASTQ files if necessary).

## Stage 2: TRIM CONSTANT REGIONS

Remove constant region sequences from read 5’ and 3’ ends using cutadapt. By default the sequences of 3' constant regions are assumed to be the reverse complement of 5' constant region sequences. Stage-specific arguments: 'cutadaptCut5First', 'cutadaptCut5Second', 'cutadaptCut3First', 'cutadaptCut3Second', 'cutadapt5First', 'cutadapt5Second', 'cutadapt3First', 'cutadapt3Second', 'cutadaptMinLength', 'cutadaptErrorRate'.

## Stage 3: ALIGN PAIRED-END READS

Align overlapping read pairs using USEARCH (paired-end cis libraries only i.e. 'paired'=T, 'transLibrary'=F) or alternatively concatenate read pairs (paired-end trans libraries only i.e. 'transLibrary'=T), and filter resulting variants according to base quality, expected number of errors and constituent read length (including those from single-end libraries i.e. 'paired'=F). Stage-specific arguments: 'usearchMinQual', 'usearchMaxee', 'usearchMinlen', 'usearchMinovlen'. Unique variant sequences are then tallied using [starcode](https://github.com/gui11aume/starcode).

## Stage 4: MERGE SAMPLE STATISTICS AND FILTER VARIANTS

Combine sample-wise variant counts and statistics to produce a unified results data.table. After aggregating counts across technical replicates, variants are processed and filtered according to user specifications:
* **4.1** For barcoded libraries, read counts are aggregated at the variant level for barcode/variant mappings specified in the variant identity file (see below). Undefined/misread barcodes are ignored.
* **4.2** Indel variants (defined as those not matching the wild-type nucleotide sequence length) are removed.
* **4.3** If internal constant region(s) are specified (lower-case letter in 'wildtypeSequence' argument), these are excised from all variants if a perfect match is found.
* **4.4** Variants with mutations inconsistent with the library design are removed (specified with 'permittedSequences' argument).
* **4.5** Variants with more substitions than specified with 'maxSubstitutions' are also removed.
* **4.6** Finally, nonsynonymous variants with synonymous substitutions in other codons are removed (if 'mixedSubstitutions'=F).

## Stage 5: CALCULATE FITNESS

Calculate fitness and error estimates for a user-specified subset of substitution variants:
* **5.1** Low count variants are removed according to user-specified soft ('fitnessMinInputCountAny', 'fitnessMinOutputCountAny') and hard ('fitnessMinInputCountAll', 'fitnessMinOutputCountAll') thresholds to minimise the impact of fake variants from sequencing errors.
* **5.2** An error model is fit to a high confidence subset of variants to determine count-based (Poisson), replicate and over-sequencing error terms.
* **5.3** Variants are aggregated at the amino acid level if the target molecule is a protein ('sequenceType'=coding).
* **5.4** Fitness and estimates of the associated error are then calculated with respect to the corresponding wild-type sequence score using the model derived in **5.3** above.
* **5.5** (*Coming soon: still in development*) Optionally improve double mutant fitness estimates for low frequency variants using a Bayesian approach that incorporates priors based on observed single mutant counts ('bayesianDoubleFitness', 'bayesianDoubleFitnessLamD', 'fitnessHighConfidenceCount', 'fitnessDoubleHighConfidenceCount').
* **5.6** In the case of a growth-rate based assay, a 'generations' column can be supplied in the experimental design file in order to normalize fitness and error estimates accordingly (see below).
* **5.7** Fitness scores are merged between replicates in a weighted manner that takes into account their respective errors.

## Output Files

* **PROJECT_DIR/PROJECT_NAME_fitness_replicates.RData** R data object with replicate (and merged) variant fitness scores and associated errors ('all_variants' data.table).
* **PROJECT_DIR/PROJECT_NAME_variant_data_merge.RData** R data object with variant counts and statistics ('variant_data_merge' data.table).
* **PROJECT_DIR/PROJECT_NAME_variant_data_merge.tsv** Tab-separated plain text file with variant counts and statistics.
* **PROJECT_DIR/PROJECT_NAME_nobarcode_variant_data_merge.tsv** Tab-separated plain text file with sequenced barcodes that were not found in the variant identity file.
* **PROJECT_DIR/PROJECT_NAME_indel_variant_data_merge.tsv** Tab-separated plain text file with indel variants.
* **PROJECT_DIR/PROJECT_NAME_rejected_variant_data_merge.tsv** Tab-separated plain text file with rejected variants (internal constant region mutants, mutations inconsistent with the library design or variants with too many substitutions).
* **PROJECT_DIR/report.html** DiMSum pipeline summary report and diagnostic plots in html format.




(Vector illustration credit: <a href="https://www.vecteezy.com">Vecteezy!</a>)
