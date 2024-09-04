This track is a collection of Nuclear mitochondrial genome segments, provided in [BED][] format.

**Notice**: Alignments reference to incompletely assembled or unmapped chromosome locations are omitted in this track.

|  Genome name |   Animal   | UCSC genome browser link  |
|--------------|------------|---------------------------|
|     hg38     |    human   |      [hg38 link][]        |
|     hg19     |    human   |      [hg19 link][]        |
|     mm39     |    mouse   |      [mm39 link][]        |
|    panTro6   | chimpanzee |      [panTro6 link][]     |
|    susScr11  |    pig     |     [susScr11 link][]     |

## Comparison with 2011 human UCSC dataset
Our track offers NUMTs both in hg19 and hg38 assemblies. The exsiting human UCSC database is based on hg19 assembly. We conducted a comparison between our track and the exsiting human UCSC database using bedtools v2.31.1. The results are shown in the following table:

|        Track            |  Total Counts  |  1bp Overlap Rate |  50% Overlap Rate  |  90% Overlap Rate  |
|-------------------------|----------------|-------------------|--------------------|--------------------|
|  2011 hg19 NUMTs track  |      766       |         -         |         -          |         -          |
|  Our hg19 NUMTs track   |      1077      |   98.7%(756/766)  |   98.3%(753/766)   |   96.6%(740/766)   |


# Method

**NUMTs are detected by the following steps:**

## Nuclear genome-mitochondrial genome comparison
We first compared the nuclear genome to the mitochondrial genome by [LAST][]. The sample commands are shown below:

    lastdb -P8 -c mitogenodb $mitogenoFASTA
    last-train -S0 --pid=70 --sample-number=0 -P8 mitogenodb $nuclearFASTA > nu2mitogeno.train
    lastal -P8 -H1 -J1 -R00 -p nu2mitogeno.train mitogenodb $nuclearFASTA > nu2mitogeno.maf

## Nuclear genome-mitochondrial protein comparison
Comparison between nuclear genome and mitochondrial protein was also completed by [LAST][], with commands:

    lastdb -P8 -q -c mitoprodb $mitoproFASTA
    last-train --codon --pid=70 --sample-number=0 -P8 mitoprodb $nuclearFASTA > nu2mitopro.train
    lastal -P8 -H1 -K1 -m500 -p nu2mitopro.train mitoprodb $nuclearFASTA > nu2mitopro.maf
    
`-P8` makes it faster by using 8 threads, with no effect on results.


## Remove nuclear ribosomal RNA regions
Before removing, we need to convert the result from maf format to [BED][] format using `maf-Bed`:

    maf-Bed nu2mitogeno.maf > nu2mitogeno.bed
    maf-Bed nu2mitopro.maf > nu2mitopro.bed


Alignments that overlapped with nuclear ribosomal RNA region were excluded respectively in the two comparisons. This is due to the fact that mitochondrial rRNA is similar to nuclear rRNA, resulting in an overestimation of ancient NUMTs counts as some mitochondrial sequences are mistakenly aligned with the nuclear rRNA regions.

This step needs [seg-suite][], so you may need to install it beforehand.

    remvrrna nu2mitogeno.bed rRNA.bed
    remvrrna nu2mitopro.bed rRNA.bed

## Merge results 
The strand information in protein search is not consistent with it in genome search, so before merging, we need to make it consistent with results in genome search. To convert, we need to know which mitochondrial DNA strand each mitochondrial protein comes from:

    lastdb -P8 -q -c mitoprodb $mitoproFASTA
    last-train -P8 --codon mitoprodb $mitogenoFASTA > mtgeno2pro.train
    lastal -P8 -D1e10 -p mtgeno2pro.train mitoprodb $mitogenoFASTA > mtgeno2pro.maf
    fix-protein-strand mtgeno2pro.maf nu2mitopro_movrrna.bed > nu2mitopro_movrrna_fix.bed

Finally, if the two alignments from the two comparisons respectively, are in the same strand and either overlap by at least 1bp or are bookended, they will be merged into a single NUMT. Alignments with a length of less than 30bp in the merged result were removed. This threshold was determined empirically.

This step needs [bedtools merge][], so you may need to install it beforehand.

    merge nu2mitogeno_movrrna.bed nu2mitopro_movrrna_fix.bed $yourspecies 

Please set `$yourspecies` to the name of the species you are looking at.


[LAST]: https://gitlab.com/mcfrith/last/-/tree/main?ref_type=heads
[BED]: https://genome.ucsc.edu/FAQ/FAQformat.html#format1
[seg-suite]: https://github.com/mcfrith/seg-suite
[bedtools merge]: https://bedtools.readthedocs.io/en/latest/content/tools/merge.html
[hg38 link]: https://genome.ucsc.edu/cgi-bin/hgTracks?db=hg38&hubUrl=https://raw.githubusercontent.com/Koumokuyou/NUMTs/main/hub.txt
[hg19 link]: https://genome.ucsc.edu/cgi-bin/hgTracks?db=hg19&hubUrl=https://raw.githubusercontent.com/Koumokuyou/NUMTs/main/hub.txt
[mm39 link]: https://genome.ucsc.edu/cgi-bin/hgTracks?db=mm39&hubUrl=https://raw.githubusercontent.com/Koumokuyou/NUMTs/main/hub.txt
[panTro6 link]: https://genome.ucsc.edu/cgi-bin/hgTracks?db=panTro6&hubUrl=https://raw.githubusercontent.com/Koumokuyou/NUMTs/main/hub.txt
[susScr11 link]: https://genome.ucsc.edu/cgi-bin/hgTracks?db=susScr11&hubUrl=https://raw.githubusercontent.com/Koumokuyou/NUMTs/main/hub.txt