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