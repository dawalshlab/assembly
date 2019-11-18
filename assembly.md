# Metagenome assembly pipeline

Created by Susanne Kraemer

## Download raw .fastq files

### 1. Download files from Nanuq
- Log into Nanuq.
- Check the boxes of your samples under the NovaSeq Read Sets tab and click _Download files_.
- Check _Download files from selected reads_.
- Uncheck everything but _Fastq R1_ and _Fastq R2_.
- Toggle between _Text file with URL links_ and _Md5 file_.
- Download (you should get a text file with the links).

### 2. Download files to the server
- Move the downloaded folder (containing _README_, _readSetLinks.txt_, _run_wget.sh_) to the server.
- Run the _run_wget.sh_ script: ```./run_wget.sh```
- Enter Nanuq username and password.
- __You should download two .fastq files and two md5 files per sample__.

### 3. Check md5 sums
- __md5 sum__: contains file size as a hexadecimal number.
- md5 sums ensure that your downloads did not get interrupted.
- Check md5 sums (on Linux): ```md5sum -c```
- Checking md5 sums will check file size and look for corresponding md5 file for each of our read files and tell you if there is a match.
- _If file is not OK_: Download again.

## Filter and trim reads

- Reads in the .fastq files still have adapter sequences attached.
- We want to remove adapters and low quality bases.
- Program: trimmomatic-0.38
- The program needs forward read file name, reverse read file name, and file with adapter sequences (you can get those from the librairies tab in Nanuq).
```shell
java –jar trimmomatic-0.38.jar PE read1.fastq.gz read2.fastq.gz
trimmed1.p.fastq.gz trimmed1.u.fastq.gz trimmed2.p.fastq.gz
trimmed2.u.fastq.gz ILLUMINACLIP: NovaSeq.fa:2:30:10 LEADING:3
TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
```
- Proceed with _trimmed1.p.fastq.gz_ and _trimmed2.p.fastq.gz_ files.
