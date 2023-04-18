# Data for insect use case

Genomes of insects were downloaded as follows:

```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/014/466/185/GCA_014466185.1_ASM1446618v1/GCA_014466185.1_ASM1446618v1_genomic.fna.gz
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/014/466/195/GCA_014466195.1_ASM1446619v1/GCA_014466195.1_ASM1446619v1_genomic.fna.gz
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/014/466/175/GCA_014466175.1_ASM1446617v1/GCA_014466175.1_ASM1446617v1_genomic.fna.gz
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/001/465/965/GCA_001465965.1_Pdom_r1.2/GCA_001465965.1_Pdom_r1.2_genomic.fna.gz
```

Genomes were extracted and softlinked in separate directories as follows (we used the provided repeat masking):

```
gunzip *.fna.gz
ln -s *.fna.gz genome.fa
```