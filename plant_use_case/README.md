# Data for plant use case

The genome of *Coix aquatica* was downloaded, extracted and softlinked from NCBI as follows:

```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/009/725/075/GCA_009725075.1_ASM972507v1/GCA_009725075.1_ASM972507v1_genomic.fna.gz
gunzip GCA_009725075.1_ASM972507v1_genomic.fna.gz
ln -s GCA_009725075.1_ASM972507v1_genomic.fna genome.fa
export GENOME=genome.fa
```

## Repeat masking

A species-specific repeat library was generated with RepeatModeler as follows:

```
export DB=Coix_aquatica
BuildDatabase -name ${DB} ${GENOME}
RepeatModeler -database ${DB} -pa 72 -LTRStruct
```

The genome was subsequently softmasked for repeats with RepeatMasker:

```
RepeatMasker -pa 72 -lib ${DB}-families.fa -xsmall ${GENOME}
```

## Running GALBA

GALBA was run with the following command:

```
galba.pl --genome=genome.fa --prot_seq=proteins.fa --threads 72
```