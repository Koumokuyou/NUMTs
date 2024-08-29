This track is a collection of Nuclear mitochondrial genome segments, provided in [BED][] format.

**Notice**: Alignments reference to incompletely assembled or unmapped chromosome locations are omitted in this track.

| Genome name  | Animal     | Source   |
|--------------|------------|----------|
| hg38         | human      | [NCBI][] |
| mm39         | mouse      | [NCBI][] |
| panTro6      | chimpanzee | [NCBI][] |
| susScr11     | pig        | [NCBI][] |

You can easily view the NUMTs in the UCSC genome browser, by clicking:

hg38:
https://genome.ucsc.edu/cgi-bin/hgTracks?db=hg38&hubUrl=https://raw.githubusercontent.com/Koumokuyou/NUMTs/main/hub.txt

mm39:
https://genome.ucsc.edu/cgi-bin/hgTracks?db=mm39&hubUrl=https://raw.githubusercontent.com/Koumokuyou/NUMTs/main/mm39/trackDb.txt

panTro6:
https://genome.ucsc.edu/cgi-bin/hgTracks?db=panTro6&hubUrl=https://raw.githubusercontent.com/Koumokuyou/NUMTs/main/panTro6/trackDb.txt

susScr11:
https://genome.ucsc.edu/cgi-bin/hgTracks?db=susScr11&hubUrl=https://raw.githubusercontent.com/Koumokuyou/NUMTs/main/susScr11/trackDb.txt

NUMTs in this track are found by the steps below:

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

## Reverse search
Next, we repeated the search using a reversed query sequence respectively in the two comparisons described above for negative control.
Take the reverse search in *Nuclear genome-mitochondrial genome comparison* for example, the sample commands are like this:

     lastal -P8 -H1 -J1 -R00 --reverse -p nu2mitogeno.train mitogenodb $nuclearFASTA > rev_nu2mitogeno.maf

The highest score obtained from the reverse search was used as a threshold to filter out alignments with score lower than those from the original search.

    filter_maf nu2mitogeno.maf rev_nu2mitogeno.maf > filtered_nu2mitogeno.maf
    filter_maf nu2mitopro.maf rev_nu2mitopro.maf > filtered_nu2mitopro.maf

## Remove nuclear ribosomal RNA regions
Before removing, we need to convert the result from maf format to [BED][] format using [maf-convert]:

    maf-convert bed -s2 filtered_nu2mitogeno.maf > filtered_nu2mitogeno.bed
    maf-convert bed filtered_nu2mitopro.maf > filtered_nu2mitopro.bed


Alignments that overlapped with nuclear ribosomal RNA region were also excluded respectively in the two comparisons. This is due to the fact that mitochondrial rRNA is similar to nuclear rRNA, resulting in an overestimation of ancient NUMTs counts as some mitochondrial sequences are mistakenly aligned with the nuclear rRNA regions.

This step needs [seg-suite][], so you may need to install it beforehand.

    remvrrna filtered_nu2mitogeno.bed rRNA.bed
    remvrrna filtered_nu2mitopro.bed rRNA.bed

## Merge results 
Finally, if the two alignments from the two comparisons respectively, are in the same orientation and either overlap by at least 1bp or are bookended, they will be merged into a single NUMT. Alignments with a length of less than 30bp in the merged result were removed. This threshold was determined empirically.

This step needs [bedtools merge][], so you may need to install it beforehand.

    merge filtered_nu2mitogeno_movrrna.bed filtered_nu2mitopro_movrrna.bed $yourspecies 

Please set `$yourspecies` to the name of the species you are looking at.


[LAST]: https://gitlab.com/mcfrith/last/-/tree/main?ref_type=heads
[BED]: https://genome.ucsc.edu/FAQ/FAQformat.html#format1
[maf-convert]: https://gitlab.com/mcfrith/last/-/blob/main/doc/maf-convert.rst?ref_type=heads
[seg-suite]: https://github.com/mcfrith/seg-suite
[bedtools merge]: https://bedtools.readthedocs.io/en/latest/content/tools/merge.html
[NCBI]: https://www.ncbi.nlm.nih.gov/genome