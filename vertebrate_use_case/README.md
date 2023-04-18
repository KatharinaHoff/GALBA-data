# Data for vertebrates use case

Genomes of whales and dolphins were downloaded from NCBI as follows:

```
# Balaenoptera bonaerensis
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/978/805/GCA_000978805.1_ASM97880v1/GCA_000978805.1_ASM97880v1_genomic.fna.gz
# Eubalaena japonica
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/004/363/455/GCA_004363455.1_EubJap_v1_BIUU/GCA_004363455.1_EubJap_v1_BIUU_genomic.fna.gz
# Inia geoffrensis
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/004/363/515/GCA_004363515.1_IniGeo_v1_BIUU/GCA_004363515.1_IniGeo_v1_BIUU_genomic.fna.gz
# Kogia breviceps
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/004/363/705/GCA_004363705.1_KogBre_v1_BIUU/GCA_004363705.1_KogBre_v1_BIUU_genomic.fna.gz
# Phocoena phocoena
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/004/363/495/GCA_004363495.1_PhoPho_v1_BIUU/GCA_004363495.1_PhoPho_v1_BIUU_genomic.fna.gz
# Platanista gangetica
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/004/363/435/GCA_004363435.1_PlaMin_v1_BIUU/GCA_004363435.1_PlaMin_v1_BIUU_genomic.fna.gz
# Ziphius cavirostris
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/004/364/475/GCA_004364475.1_ZipCav_v1_BIUU/GCA_004364475.1_ZipCav_v1_BIUU_genomic.fna.gz
```
Each file was stored in a separate folder and uncompressed with `gunzip` and softlinked as follows:

```
gunzip *.fna.gz
ln -s *.fna.gz genome.fa
export GENOME=genome.fa
```

## Repeat masking

Species-specific repeat libraries were generated with RepeatModeler as follows (comparable to the procedure for vertebrates described in the [EukSpecies-BRAKER2](https://github.com/gatech-genemark/EukSpecies-BRAKER2):

```
exprot DB=species_name # replace by actual species name
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

## Running GALBA

GALBA was run with the following command:

```
galba.pl --genome=genome.fa --prot_seq=proteins.fa --threads 72
```