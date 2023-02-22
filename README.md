# De Novo Assembly

##### Author: Iñaki Sasiain Casado #####

##### Date: 22/02/2023

##### BINP29

## PROCEDURE

1. Create a conda environment for the workflow and download the required software.

```bash
conda create -n sra-tools;
conda activate sra-tools;
conda install -c bioconda sra-tools=3.0.0;

conda create -n fastqc;
conda activate fastqc;
conda install -c bioconda fastqc=0.11.9;

conda create -n hifiasm;
conda activate hifiasm;
conda install -c bioconda hifiasm=0.18.7;

```

2. Download the required data from NCBI Sequence Reads Archive and store the data into a the Data directory, which is excluded from the git repository.

 ```bash
mkdir Data;
cd Data;
conda activate sra-tools;
fastq-dump SRR13577846;
mv ./SRR13577846.fastq ./long_reads.fastq;

cd ../;
echo "Data/" > .gitignore;
 ```

3. Quality checking of the reads using FastQC.

```bash
mkdir 01_FastQC;
cd 01_FastQC;
conda activate fastqc;
time fastqc ../Data/long_reads.fastq -o .;
```

>The time required for running the fastqc program was 31,209 seconds.The per base sequence content shows that the nucleotide content becomes unstable at the end of the reads. Considering, that the amount of reads that exceed that number of bases is really low the observed results can be explained. The measured GC content of the reads suggests the presence of contaminant sequences.

5. Assembly using hifiasm. Fasta files were created from the .gfa files.

```bash
mkdir ../02_Hifiasm;
cd ../02_Hifiasm;
conda activate hifiasm;
time hifiasm ../Data/long_reads.fastq;

ls *.gfa | while read name; do newname=$(echo ${name} | sed 's/.gfa/.fasta/'); awk '/^S/{print ">"$2"\n"$3}' ${name} > ${newname}; done;
```

>The time required for running the hifi assembler was 48 minutes and 47 seconds.