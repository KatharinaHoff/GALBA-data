# Data for plant use case

The genome of *Coix aquatica* was downloaded, extracted and softlinked from NCBI as follows:

```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/009/725/075/GCA_009725075.1_ASM972507v1/GCA_009725075.1_ASM972507v1_genomic.fna.gz
gunzip GCA_009725075.1_ASM972507v1_genomic.fna.gz
ln -s GCA_009725075.1_ASM972507v1_genomic.fna genome.fa
export GENOME=genome.fa
```

## Repeat masking

Species-specific repeat libraries were generated with RepeatModeler as follows (comparable to the procedure for vertebrates described in the [EukSpecies-BRAKER2](https://github.com/gatech-genemark/EukSpecies-BRAKER2):

```
export DB=Coix_aquatica
BuildDatabase -name ${DB} ${GENOME}
RepeatModeler -database ${DB} -pa 72 -LTRStruct
```

Genomes were subsequently softmasked for repeats with RepeatMasker and tandem repeats with TRF as follows:

```
RepeatMasker -pa 72 -lib ${DB}-families.fa -xsmall ${GENOME}

trf $GENOME 2 7 7 80 10 50 500 -d -m -h

parseTrfOutput.py ${GENOME}.2.7.7.80.10.50.500.dat --minCopies 1 --statistics STATS > ${GENOME}.2.7.7.80.10.50.500.raw.gff

sort -k1,1 -k4,4n -k5,5n ${GENOME}.2.7.7.80.10.50.500.raw.gff > sorted

bedtools merge -i sorted | awk ’BEGIN{OFS="\t"} {print $1,"trf","repeat",$2+1,$3,".",".",".","."}’ > ${GENOME}.2.7.7.80.10.50.500.merged.gff

bedtools maskfasta -fi ${GENOME}.masked -bed ${GENOME}.2.7.7.80.10.50.500.merged.gff -fo ${GENOME}.combined.masked -soft
```