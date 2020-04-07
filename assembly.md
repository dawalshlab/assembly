# Metagenome assembly pipeline

Created by Susanne Kraemer; transcribed and edited by Rebecca Garner

## Table of contents

* [Download raw .fastq files](#download-raw-fastq-files)
  1. [Download files from Nanuq](#1-download-files-from-nanuq)
  2. [Download files to the server](#2-download-files-to-the-server)
  3. [Check md5 sums](#3-check-md5-sums)

* [Trim reads](#trim-reads)

* [Assemble metagenomes](#assemble-metagenomes)

* [Reconstruct full-length ribosomal genes](#reconstruct-full-length-ribosomal-genes) (_optional_)

* [Map reads](#map-reads)

* [Submit assemblies to IMG](#submit-assemblies-to-img)

## Download raw .fastq files

### 1. Download files from Nanuq
- Log into [Nanuq](https://genomequebec.mcgill.ca/nanuqAdministration/ "Nanuq").
- Under the _NovaSeq Read Sets_ tab, check the boxes of your samples and click _Download Read Files_.
- Check _Download files from selected reads_.
  - __Type of Download__: Check _Text file with URL links_.
  - __Type of Files to Download__: Uncheck everything but _Fastq R1_ and _Fastq R2_.
- Download (downloaded folder contains _Readme.txt_, _readSetLinks.txt_, and _run_wget.sh_).

### 2. Download files to the server
- Move the downloaded folder and MD5 file to the server.
- Run the _run_wget.sh_ script: ```sh run_wget.sh```
- Enter __Nanuq__ username and password.
  
This will start the download of two .fastq files (forward _\_R1_ and reverse _\_R2_ reads) and two corresponding md5 files __per sample__.

### 3. Check md5 sums
__md5__ files contain file size as a hexadecimal number which can be used to verify file integrity (e.g., make sure your downloads are intact). Use ```md5sum``` to ensure that your downloads did not get interrupted. ```md5sum``` will check file size and look for the corresponding md5 file for each of your read files and evaluate if there is a match.
  
- Check md5 sums: ```md5sum -c *.md5```
- If file is not _OK_ (i.e. _FAILED_): Redownload .fastq and associated files from Nanuq.

## Trim reads

The raw reads in the .fastq files have adaptor sequences attached. We want to remove adaptors and low quality bases. The trimming program (```trimmomatic```) requires 3 files: forward reads .fastq (zipped), reverse reads .fastq (zipped), and a .fasta file containing the adaptor sequences.  The adaptor sequences are available in Nanuq under the _NovaSeq Read Sets_ tab (_Adaptor_ column).
  
- Create a .fasta file containing the adaptor sequences.  E.g.,
```
>PrefixPE/1
AGATCGGAAGAGCACACGTCTGAACTCCAGTCAC
>PrefixPE/2
AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT
```
- Trim adaptor sequences, low quality bases, and ultrashort reads with [trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic "Trimmomatic") version 0.38:

```shell
java –jar trimmomatic-0.38.jar PE R1.fastq.gz R2.fastq.gz
R1_p_trimmed.fastq.gz R1_u_trimmed.fastq.gz R2_p_trimmed.fastq.gz R2_u_trimmed.fastq.gz
ILLUMINACLIP:NovaSeq.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
```
- 
  - _R1.fastq.gz_ and _R2.fastq.gz_ are the raw, zipped .fastq files downloaded from Nanuq.
  - _R1_p_trimmed.fastq.gz_ and _R2_p_trimmed.fastq.gz_ are the outputted trimmed, _paired_ reads.
  - _R1_u_trimmed.fastq.gz_ and _R2_u_trimmed.fastq.gz_ are the outputted trimmed, _unpaired_ reads.
  - _NovaSeq.fa_ is the .fasta file containing the adaptor sequences.
- Proceed with trimmed reads in _R1_p_trimmed.fastq.gz_ and _R2_p_trimmed.fastq.gz_ files.

## Assemble metagenomes

- Assemble a metagenome relatively quickly with [MEGAHIT](https://github.com/voutcn/megahit "MEGAHIT") version 1.0.6.

```shell
megahit -1 PATH/R1_p_trimmed.fastq.gz -2 PATH/R2_p_trimmed.fastq.gz
--k-list 23,43,63,83,103,123 -o PATH/OUTPUT_DIRECTORY/ --verbose
```

- You can create a combined assembly (co-assembly) by listing reads from separate metagenomes separated by commas.

```shell
megahit -1 PATH/METAGENOME1_R1_p_trimmed.fastq.gz,PATH/METAGENOME2_R1_p_trimmed.fastq.gz,PATH/METAGENOME3_R1_p_trimmed.fastq.gz
-2 PATH/METAGENOME1_R2_p_trimmed.fastq.gz,PATH/METAGENOME2_R2_p_trimmed.fastq.gz,PATH/METAGENOME3_R2_p_trimmed.fastq.gz
--k-list 23,43,63,83,103,123 -o PATH/OUTPUT_DIRECTORY/ --verbose
```

## Reconstruct full-length ribosomal genes

This is an _optional_ step to reconstruct full-length SSU rRNA genes from unassembled metagenomes with [EMIRGE](https://github.com/csmiller/EMIRGE "EMIRGE").

- Unzip reverse (R2) reads using ```zcat R2_p_trimmed.fastq.gz > R2_p_trimmed.fastq```
- Execute the EMIRGE command:
```shell
emirge.py PATH/OUTPUT_DIRECTORY -1 PATH/R1_p_trimmed.fastq.gz -2 PATH/R2_p_trimmed.fastq
-f PATH/SILVA_132_SSURef_Nr99_tax_silva_trunc.ge1200bp.le2000bp.0.97.fixed.fasta
-b PATH/SILVA_132_SSURef_Nr99_tax_silva_trunc.ge1200bp.le2000bp.0.97.fixed
-l 151 -i 500 -s 500 --phred33
```
- Execute EMIRGE rename fasta command: 

```shell
emirge_rename_fasta.py PATH/OUTPUT_DIRECTORY/iter.40 > PATH/OUTPUT_DIRECTORY/SAMPLE_renamed.fasta
```

## Map reads

Read mapping can be perfomed with either BBMap or Burrows-Wheeler Aligner (BWA).

## Submit assemblies to IMG

- Format BBMap coverage file (_covstats.txt_) for IMG GOLD submission in R:
```r
# Load library
library(tidyverse)

# Import covstats.txt
(covstats <- read_tsv("PATH/covstats.txt", col_names = TRUE) %>%
  rename(ID = `#ID`) %>%
  select(ID, Avg_fold))

# Write new file
write_tsv(covstats, path = "PATH/OUTPUT_DIRECTORY/avgfold.txt")
```