# Data of 14 reference genomes used for measuring accuracy of GALBA

We used data prepared as described in detail at https://github.com/gatech-genemark/EukSpecies-BRAKER2 (reference annotation and repeat masked genomes) and https://github.com/gatech-genemark/BRAKER2-exp (VARUS sampled RNA-Seq reads for generating reliable gene set) of 

   * *Arabidopsis thaliana*
   * *Bombus terrestris*
   * *Caenorhabditis elegans*
   * *Drosophila melanogaster*
   * *Rhodnius prolixus*
   * *Parasteatoda tepidariorum*
   * *Populus trichocarpa*
   * *Medicago truncatula*
   * *Solanum lycopersicum*
   * *Xenopus tropicalis*
   
Further, we used data prepared as described in detail at https://github.com/gatech-genemark/GeneMark-ETP-exp (reference annotation, repeat masked genomes and reliable gene sets) of

   * *Danio rerio*
   * *Gallus gallus*
   * *Mus musculus*

The reference proteomes are listed in Supplementary Table S1 of the GALBA Preprint at https://www.biorxiv.org/content/10.1101/2023.04.10.536199v1.full.pdf . After download, concatenated the respective reference proteome files (except for the single-reference-proteome runs) and modified the FASTA headers with the following command:

```
# in folder for reference proteomes of a particular species
zcat *.faa.gz > prots.fa
simplifyFastaHeaders.pl prots.fa prot proteins.fa header.map
```

simplifyFastaHeaders.pl is part of AUGUSTUS at https://github.com/Gaius-Augustus/Augustus/tree/master/scripts .

For single-single-reference-proteome run, headers were modified without concatenation.

### Miniprothint scoring figure

Figure 3 (a scatterplot of introns predicted by miniprot, characterized by miniprothint-derived IMC and IBA scores) was generated from the reference annotation (the source described above) and `miniprot.gff` (generated during a GALBA run) in the following way:

```bash
collapseGff.py miniprot.gff > miniprotCollapsed.gff
# Sorting by coordinates ensures the FPs and TPs are well-mixed
sort -k1,1 -k4,4n -k5,5n miniprotCollapsed.gff > miniprotCollapsedSorted.gff
visualizeMiniprothint.py miniprotCollapsedSorted.gff annot.gtf figure.pdf --ylim 25
```

Both `collapseGff.py` and `visualizeMiniprothint.py` are available at https://github.com/tomasbruna/miniprothint.

## Running GALBA

GALBA was executed and evaluated as follows:

```
galba.pl --genome=genome.fa --prot_seq=proteins.fa --threads 72 --annot=annot.gtf --pseudo=pseudo.gff3
```
The number of threads varied between runs, depending on HPC node availability.

## Running BRAKER2

BRAKER2 was executed and evaluated with singularity as follows:

```
singularity exec braker3.sif braker.pl --genome=genome.fa --prot_seq=proteins.fa --threads 72 --annot=annot.gtf --pseudo=pseudo.gff3
```

The number of threads varied between runs, depending on HPC node availability.

## Running TSEBRA

TSEBRA was executed as follows:

```
tsebra.py -g braker.gtf --keep_gtf galba.gtf \
    -e braker_hintsfile.gff,galba_hintsfile.gff -c default.cfg -o tsebra.gtf
```

## Running FunAnnotate

FunAnnotate was applied as follows:

```
# only once, to get the singularity container
singularity pull docker://nextgenusfs/funannotate

export GENEMARK_PATH=/path/to/GeneMark-ES-ET-EP_v4.71_lic/gmes_funannotate

species="name of species"
buscoSeedSpecies="name of seed species"
buscodb="name of busco db"
genomepath="/path/to/genome.fasta.masked"
protpath="/path/to/proteins.fa"

# calculateGenomeSizeFromFasta.pl adds up the length of all sequences in a fasta
genomeSize=$(perl ~/calculateGenomeSizeFromFasta.pl $genomepath)
maxIntronLen_f=$(echo "3.6 * sqrt($genomeSize)" | bc -l)
maxIntronLen=$(printf "%.0f" "$maxIntronLen_f")

mkdir -p fun tmp
singularity run funannotate_latest.sif funannotate predict \
    --input $genomepath --out fun --species $species \
    --busco_seed_species $buscoSeedSpecies --busco_db $buscodb \
    --organism other --protein_evidence $protpath \
    --max_intronlen $maxIntronLen --cpus 72 --tmpdir tmp --no-progress \
    --repeats2evm
```

For accuracy evaluation, the gff3 output of FunAnnotate was converted from gff3 to gtf format using gff3_to_gtf.pl from GeneMark-ET, and with compute_accuracies.sh from BRAKER:

```
gff3_to_gtf.pl funannotate.gff3 funannotate.gtf
compute_accuracies.sh annot.gtf pseudo.gff3 funannotate.gtf gene trans cds
```

FunAnnotate sometimes modifies sequence names in the output, automatically. We had to revert these sequence name changes to match the reference annotation. This was in particular the case for *Medicago truncatula*:

```
cat funannotate.gtf | perl -pe 's/Mrun/Mtrun/' > funannotate.f.gtf
mv funannotate.f.gtf funannotate.gtf
```