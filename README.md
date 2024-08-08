This track shows Nuclear mitochondrial genome segments in human genome(Assembly GRCh38.p14). It show alignments of the nuclear genome to mitochondrial genome.

**Notice**: Alignments reference to incompletely assembled or unmapped chromosome locations are omitted in this track.

# NUMTs search
NUMTs in this track are found by the steps below:

## Nuclear genome-mitochondrial genome comparison
We first compared nuclear genome to mitochondrial genome by [LAST][]. The sample commands are shown below:

    lastdb -P8 -c mitogenodb $mitogenoFASTA
    last-train -S0 --pid=70 --sample-number=0 -P8 mitogenodb $nuclearFASTA > nu2mitogeno.train
    lastal -P8 -D$Dopt -J1 -R00 -p nu2mitogeno.train mitogenodb $nuclearFASTA > nu2mitogeno.maf

## Nuclear genome-mitochondrial protein comparison
Comparison between nuclear genome and mitochondrial protein was also completed by [LAST][], with commands:

    lastdb -P8 -q -c mitoprodb $mitoproFASTA
    last-train --codon --pid=70 --sample-number=0 -P8 mitoprodb $nuclearFASTA > nu2mitopro.train
    lastal -P8 -D$Dopt -K1 -m500 -p nu2mitopro.train mitoprodb $nuclearFASTA > nu2mitopro.maf

**Notice**: For `$Dopt` one can set it to adjust the sensitivity of results in this search. See [E-value options][] for more details. In this track, `$Dopt` was set the same as the size of genome in $nuclearFASTA.

## Reverse search
Next, we repeated the search using a reversed query sequence respectively in the two comparison described above for negative control. Take the reverse search in *Nuclear genome-mitochondrial genome comparison* for example, the sample commands are like:

    lastal -P8 -D$Dopt -J1 -R00 -p nu2mitogeno.train mitogenodb $revnuclearFASTA > rev_nu2mitogeno.maf 

The lowest e-value obtained from the reverse search was used as a threshold to filter out alignments with e-values higher than those from the original search.

    filter_maf nu2mitogeno.maf rev_nu2mitogeno.maf > filtered_nu2mitogeno.maf
    filter_maf nu2mitopro.maf rev_nu2mitopro.maf > filtered_nu2mitopro.maf

## Remove nuclear ribosomal RNA regions
Before the removing, we need to convert the result from maf format to [BED] format using [maf-convert]:

    maf-convert bed filtered_nu2mitogeno.maf > filtered_nu2mitogeno.bed
    maf-convert bed filtered_nu2mitopro.maf > filtered_nu2mitopro.bed


Alignments that overlapped with nuclear ribosomal RNA region were also excluded respectively in the two comparison. This is due to the fact that mitochondrial rRNA is similar to nuclear rRNA, resulting in an overestimation of ancient NUMTs counts as some mitochondrial sequences are mistakenly aligned with the nuclear rRNA regions.

    remvrrna filtered_nu2mitogeno.bed rRNA.bed
    remvrrna filtered_nu2mitopro.bed rRNA.bed

## Merge results 
Finally, the results in the two comparison were merged, and alignments with a length less than 30bp in the merged result were removed. This threshold was determined empirically.

    merge filtered_nu2mitogeno_movrrna.bed filtered_nu2mitopro_movrrna.bed $yourspecies 
    
set `$yourspecies` to the name of species you are looking at.
[LAST]: https://gitlab.com/mcfrith/last/-/tree/main?ref_type=heads
[E-value options]: https://gitlab.com/mcfrith/last/-/blob/main/doc/lastal.rst?ref_type=heads
[BED]: https://genome.ucsc.edu/FAQ/FAQformat.html#format1
[maf-convert]: https://gitlab.com/mcfrith/last/-/blob/main/doc/maf-convert.rst?ref_type=heads
