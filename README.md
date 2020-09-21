# SARS_CoV-2_Zequencer
## introduction

Dave O'Connor's lab has had several studies where reproducible viral sequencing is important. He previouslyput together a read mapping and variant calling tool, called Zequencer, that he implemented in Geneious Pro. This is convenient for users not comfortable in the command line. But it is difficult to run in a reproducible way. Zequencer 2017 is a command-line tools that masks most of the complexity of viral sequencing from users while nonetheless generating results that can be reproduced exactly. In the initial implementation, Zequencer 2017 contains three separate but related tools. 

Here I have repurposed this tool to analyze SARS-CoV-2 tiled amplicon sequencing data.

## dependencies

The Zequencer folder contains binaries for necessary processing applications compiled for OSX 10.12:

+ bbmap_36.86
+ samtools
+ seqtk
+ snpEff

Zequencer is written in Python3 and uses dependencies by running the following docker container 
 `docker run -it -v $(pwd):/scratch -w /scratch gkmoreno/sars2_zequencer:v2 /bin/bash`
 
## running zequencer 
Step 1: Download the Docker Image

+ Now that you are in the directory with your data of interest, then you need to pull the docker image to your computer. The Docker Image contains all the information needed to run Zequencer.

[Note: If you haven't downloaded Docker to your computer, you should do this from Docker.com]

+ To pull the docker image, just run the following in your terminal window:

```
docker pull gkmoreno/sars2_zequencer:v2
```

+ This will pull the image to Docker that is on your computer. It will take a bit of time, as there are a few Gb of data.  It does not matter the directory in which you are located to do the Docker pull, as long as you are NOT located inside a Docker container.  


Step 2: Moving to the correct directory location
+ Before you run the Docker image to open the container, you need to navigate to the directory with your data in it. This includes your R1 and R2 FASTQ reads and a short-amplicon-ref FASTA file, if you need to map to that reference


Step 3: Run the Docker image to open the container.
+ By running the docker image in your working directory, you will put your current directory into the Docker container.  This is the KEY POINT - you will now be able to run your data files in the Docker container.  You can do that by running the following in your terminal window:

```bash
docker run --user $(id -u):$(id -g) \
-it -v $(pwd):/scratch -w /scratch \
gkmoreno/sars2_zequencer:v2 \
/bin/bash
```

### running Zequencer without normalzing across a,plicon  

You have a set of sequence data that was generated by sequencing amplicons of a pre-defined length.  This could be running the ARTIC Network SARS-CoV-2 multiplex PCR protocol, or it could be that you have generated three amplicons from one vRNA sample and then sequenced those three amplicons. 

To invoke the tools, Dave built a snakemake file that can interpret a set of commands and perform the necessary processing steps.  Those steps include running up to three python scripts with the previous names:

+ prepareNcbiReference.py
+ normalizeShortAmpliconCoverage.py
+ mapReadsCallVariants.py
 
Once you are in your docker container in the correct working directory, then you need to run the snakemake file using the following set of commands.

Here is the set of commands with no defined details:

```bash
snakemake \
--snakefile /zequencer/zequencer.smk \
--config \
r1_fastq= \  (file containing R1 reads)
r2_fastq= \  (file containing R2 reads)
sample_name= \ (name of output file - needs to have a LETTER in it or it will think it is an integer)
ncbi_accession= \ (Genbank accession number)
downsampling_fasta= \ (reference fasta file with short amplicons)
reads_per_amplicon=1000 \ (number of reads to extract mapping to each amplicon)
reads_per_sample= \ (number of total reads to extract, if reads_per_amplicon is left empty)
minimum_variant_percentage= \ (min percent of variants to include in vcf)
minimum_read_length= (min length of a read so you get rid of small junk)
```
Here is an example set of commands to use if you **do not** have a reference fasta file for downsampling and you have the fastq files located WITHIN the directory where you opened the Docker container

```bash
snakemake \
--snakefile /zequencer/zequencer.smk \
--config \
r1_fastq=50-rep1_R1.fastq.gz \
r2_fastq=50-rep1_R2.fastq.gz \
sample_name=50-rep1 \
ncbi_accession=MN908947.3 \
downsampling_fasta= \
reads_per_amplicon=1000 \
reads_per_sample= \
minimum_variant_percentage=0.05 \
minimum_read_length=100
```

### running the Zequencer with a pre-determined reference file of amplicon sequences 

You have a set of sequence data that was generated by sequencing amplicons of a pre-defined length.  This could be running the ARTIC Network SARS-CoV-2 multiplex PCR protocol, or it could be that you have generated three amplicons from one vRNA sample and then sequenced those three amplicons. 

To invoke the tools, Dave built a snakemake file that can interpret a set of commands and perform the necessary processing steps.  Those steps include running up to three python scripts with the previous names:

+ prepareNcbiReference.py
+ normalizeShortAmpliconCoverage.py
+ mapReadsCallVariants.py
 
Once you are in your docker container in the correct working directory, then you need to run the snakemake file using the following set of commands.

Here is the set of commands with no defined details:

```bash
snakemake \
--snakefile /zequencer/zequencer.smk \
--config \
r1_fastq= \  (file containing R1 reads)
r2_fastq= \  (file containing R2 reads)
sample_name= \ (name of output file - needs to have a LETTER in it or it will think it is an integer)
ncbi_accession= \ (Genbank accession number)
downsampling_fasta= \ (reference fasta file with short amplicons)
reads_per_amplicon=1000 \ (number of reads to extract mapping to each amplicon)
reads_per_sample= \ (number of total reads to extract, if reads_per_amplicon is left empty)
minimum_variant_percentage= \ (min percent of variants to include in vcf)
minimum_read_length= (min length of a read so you get rid of small junk)
```
Here is an example set of commands to use if you have a reference fasta file for downsampling and you have the fastq files located WITHIN the directory where you opened the Docker container

```bash
snakemake \
--snakefile /zequencer/zequencer.smk \
--config \
r1_fastq=50-rep1_R1.fastq.gz \
r2_fastq=50-rep1_R2.fastq.gz \
sample_name=50-rep1 \
ncbi_accession=MN908947.3 \
downsampling_fasta=amplicons.fasta \
reads_per_amplicon=1000 \
reads_per_sample= \
minimum_variant_percentage=0.05 \
minimum_read_length=100
```
