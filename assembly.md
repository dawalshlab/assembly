# Metagenome assembly pipeline

Created by Susanne Kraemer; transcribed and edited by Rebecca Garner

## Table of contents

* [Download raw .fastq files](#download-raw-fastq-files)
  1. [Download files from Nanuq](#1-download-files-from-nanuq)
  2. [Download files to the server](#2-download-files-to-the-server)
  3. [Check md5 sums](#3-check-md5-sums)

* [Trim reads](#trim-reads)

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
- Run the _run_wget.sh_ script: ```./run_wget.sh```
- Enter __Nanuq__ username and password.
  
This will start the download of two .fastq files (forward _\_R1_ and reverse _\_R2_ reads) and two corresponding md5 files __per sample__.

### 3. Check md5 sums
__md5__ files contain file size as a hexadecimal number which can be used to verify file integrity (e.g., make sure your downloads are intact). Use ```md5sum``` to ensure that your downloads did not get interrupted. ```md5sum``` will check file size and look for the corresponding md5 file for each of your read files and evaluate if there is a match.
  
- Check md5 sums: ```md5sum -c```
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
- Trim adaptor sequences, low quality bases, and ultrashort reads with [trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic "Trimmomatic")-0.38:

```shell
java –jar trimmomatic-0.38.jar PE READS_R1.fastq.gz READS_R2.fastq.gz
trimmed1.p.fastq.gz trimmed1.u.fastq.gz trimmed2.p.fastq.gz
trimmed2.u.fastq.gz ILLUMINACLIP:NovaSeq.fa:2:30:10 LEADING:3
TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
```
- 
  - _READS_R1.fastq.gz_ and _READS_R2.fastq.gz_ are the raw, zipped .fastq files downloaded from Nanuq.
  - _trimmed1.p.fastq.gz_ and _trimmed2.p.fastq.gz_ are the outputted trimmed, _paired_ reads.
  - _trimmed1.u.fastq.gz_ and _trimmed2.u.fastq.gz_ are the outputted trimmed, _unpaired_ reads.
  - _NovaSeq.fa_ is the .fasta file containing the adaptor sequences.
- Proceed with _trimmed1.p.fastq.gz_ and _trimmed2.p.fastq.gz_ files.
