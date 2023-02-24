# De Novo Assembly

##### Author: Iñaki Sasiain Casado #####

##### Date: 22/02/2023

##### BINP29

## INTRODUCTION

De novo assembly consists of the construction of a genome based only on the reads obtained in the sequencing, i.e. without a refernce genome. Several bioinformatic software have been created to perform this task, but their procedure, and therefore, output is different. The aim of this work is to analyse the difference in the performace of three de novo assemblers, Hifiasm (v0.18.7), Canu (v5.4.5) and Flye (v2.9.1) analysing the quality of the assembles of a *Saccharomyces cerevisiae* genome using Quast (v5.0,2) and BUSCO (v5.4.4).

## PROCEDURE

1. Create conda environments for the workflow and install the required software.

```bash
#Creating an environment and installing sra-tools
conda create -n sra-tools;
conda activate sra-tools;
conda install -c bioconda sra-tools=3.0.0;

#Creating an environment and installing fastqc
conda create -n fastqc;
conda activate fastqc;
conda install -c bioconda fastqc=0.11.9;

#Creating an environment and installing hifiasm
conda create -n hifiasm;
conda activate hifiasm;
conda install -c bioconda hifiasm=0.18.7;

#Creating an environment and installing canu
conda create -n canu;
conda activate canu;
conda install -c conda-forge gnuplot=5.4.5;
conda install -c bioconda canu=2.2;

#Creating an environment and installing flye
conda create -n flye;
conda activate flye;
conda install -c bioconda flye=2.9.1;

#Creating an environment and installing quast
conda create -n quast;
conda activate quast;
conda install -c bioconda quast=5.0.2;

#Creating an environment and installing busco
conda create -n busco;
conda activate busco; 
conda install -c conda-forge -c bioconda busco=5.4.4;
```

2. Download the required data from NCBI Sequence Reads Archive and store the data into a the Data directory.

 ```bash
#Creating a directory for the Data
mkdir Data;
cd Data;
conda activate sra-tools;

#Downloading and renaming the file.
fastq-dump SRR13577846;
mv ./SRR13577846.fastq ./long_reads.fastq;
```

3. Quality checking of the reads using FastQC.

```bash
#Creating a directory for the FastQC output
mkdir 01_FastQC;
cd 01_FastQC;

#Running fastqc and measuring the required time 
conda activate fastqc;
time fastqc ../Data/long_reads.fastq -o .;
```

>The time required for running the fastqc program was 31,209 seconds.The per base sequence content shows that the nucleotide content becomes unstable at the end of the reads. Considering, that the amount of reads that exceed that number of bases is really low the observed results can be explained. The measured GC content of the reads suggests the presence of contaminant sequences.

5. Assembly using hifiasm. Fasta files were created from the .gfa files.

```bash
#Creating a directory for running Hifiasm
mkdir ../02_Hifiasm;
cd ../02_Hifiasm;

#Running Hifiasm and measuring the required time 
conda activate hifiasm;
time hifiasm -t 128 ../Data/long_reads.fastq;

#Creating .fasta files from the output .gfa files.
ls *.gfa | while read name; do newname=$(echo ${name} | sed 's/.gfa/.fasta/'); awk '/^S/{print ">"$2"\n"$3}' ${name} > ${newname}; done;
```

>The time required for running the hifi assembler was 48 minutes and 47 seconds.

6. Assembly using canu. The approximate genome size was assumed to be 12 Mb.
```bash
#Creating a directory for running Canu
mkdir ../03_Canu;
cd ../03_Canu;

#Running Canu and measuring the required time 
conda activate canu;
time canu -p canu -d . -genomeSize=12m -pacbio-hifi -maxThreads=10 ../Data/long_reads.fastq;
```
>The time required for running canu assembler was 12 minutes and 26 seconds.

7. Assembly using Flye.
```bash
#Creating a directory for running Flye
mkdir ../04_Flye;
cd ../04_Flye;

#Running Flye and measuring the required time 
conda activate flye;
time flye -t 10 --pacbio-hifi ../Data/long_reads.fastq --out-dir .;
```
>The time required for running flye assembler was 20 minutes and 25 seconds.

8. Getting the general quality statistics of the different assemblies using quast. The statistics were calculated using a Saccharomyces cerevisiae reference genome
```bash
#Downloading the refernce S.cerevisiae genome, unzipping and renaming it.
cd ../Data;
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/146/045/GCF_000146045.2_R64/GCF_000146045.2_R64_genomic.fna.gz;
gunzip GCF_000146045.2_R64_genomic.fna.gz;
mv ./GCF_000146045.2_R64_genomic.fna ./sac_cer_reference.fna;

#Creating a directory for running Quast
mkdir ../05_Qast;
cd ../05_Quast;

#Running Quast
conda activate quast;
quast -o . -t 10 -r ../Data/sac_cer_reference.fna ../02_Hifiasm/hifiasm.asm.bp.p_ctg.fasta ../03_Canu/canu.contigs.fasta ../04_Flye/assembly.fasta;
```

9. Checking the quality of the assemblies using BUSCO.
```bash
#Creating a directory for running BUSCO
mkdir ../06_Busco;
cd ../06_Busco;

#Copying the fasta files to analyse into the 06_Busco/fasta_to_analyse directory
mkdir fasta_to_analyse;
cp ../02_Hifiasm/hifiasm.asm.bp.p_ctg.fasta ../03_Canu/canu.contigs.fasta ../04_Flye/assembly.fasta ./fasta_to_analyse;
mv ./fasta_to_analyse/assembly.fasta ./fasta_to_analyse/flye.assembly.fasta;

#Running Busco
conda activate busco;
busco -c 10 -i ./fasta_to_analyse -l saccharomycetes_odb10 -o output -m genome;
```
## RESULTS AND DISCUSSION



| Assembler | Running time | Contigs | Quast:  Largest contig | Quast:  Total length | Quast: N50 | Quast: NG50 | Quast: Missasembles | BUSCO: Complete genes | BUSCO: Fragmented genes | BUSCO: Missing genes | BUSCO: Total genes |
|-----------|--------------|---------|------------------------|----------------------|------------|-------------|---------------------|-----------------------|-------------------------|----------------------|--------------------|
| Hifiasm   | 48min 47sec  |      49 |                1506376 |             12730920 |     809047 |      809047 |                 124 |                  2129 |                       2 |                    6 |               2137 |
| Canu      | 12min 26sec  |      96 |                1506339 |             13150776 |     778969 |      808829 |                 114 |                  2129 |                       2 |                    6 |               2137 |
| Flye      | 20min 25sec  |      26 |                1497679 |             12075583 |     913849 |      913849 |                  87 |                  2129 |                       2 |                    6 |               2137 |
