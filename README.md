# De Novo Assembly

##### Author: Iñaki Sasiain Casado #####
##### Date: 22/02/2023
##### BINP29

## PROCEDURE

1. Create a conda environment for the workflow and download the required software.

```bash
conda create -n De_novo;
conda activate De_novo;
conda install -c bioconda sra-tools=3.0.0;
```

2. Download the required data from NCBI Sequence Reads Archive and store the data into a the Data directory, which is excluded from the git repository. 

 ```bash
mkdir Data;
cd Data; 
fastq-dump SRR13577846;
mv ./SRR13577846.fastq ./long_reads.fastq;

cd ../;
echo "Data/" > .gitignore
 ```