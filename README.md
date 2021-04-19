# PROCRO_genome

by Daniel J leite

drdanieljleite@gmail.com

_________________________________

Description
===========
This repository contains the assembly and annotation pipeline and the annotation for the genome of the polyclad flatworm _Prostheceraeus crozeri_. For futher details please see the publication XXX and if you have questions feel free to email me or the correspondence in the paper. All details on software versions can be found in the paper.

The annotation and fasta files:
* ```PROCRO.gtf        # GTF annotation```
* ```PROCRO.gff3       # GFF3 annotation```
* ```PROCRO_cds.fasta  # cds fasta```
* ```PROCRO_aa.fasta   # animo acid annotation```

The main pipeline commands contain
* ASSEMBLY (FLYE assembly, NextPolish polishing, Purge_dups haplotype associated deduplication)
* REPEAT MASKING (Repeat modeling and masking)
* ANNOTATION (using BRAKER)

_________________________________

## ASSEMBLY

### _Assembly_
Flye was used for assembly of the PacBio reads.

```
flye \
  --pacbio-raw ${READ1} ${READ2} \
  -g 2.53g \
  -o ${GENOME_OUT} \
  -t 40 \
  -i 1 \
  --asm-coverage 75 \
  -m 8000 \
```
### _Polish_
Polishing with NextPolish requires file of file names for long (```lgs.fofn```) and short reads (```sgs.fofn```), and a config file (```run.cfg```). The ```run.cfg``` file was formatted as below, with .

```
[General]
job_type = local
job_prefix = <DIR>
task = best
rewrite = yes
rerun = 3
parallel_jobs = 4
multithread_jobs = 8
genome = <GENOME>
genome_size = 2250000000
workdir = ./polish
polish_options = -p {multithread_jobs}

[sgs_option]
sgs_fofn = ./sgs.fofn
sgs_options = -max_depth 35 -minimap2

[lgs_option]
lgs_fofn = ./lgs.fofn
lgs_options = -min_read_len 5k -max_read_len 300k -max_depth 60
lgs_minimap2_options = -x map-pb -t {multithread_jobs}
```

This was run as standard using

```NextPolish run.cfg```

### _De-duplication_

Purge_dups was used to remove duplicate contigs that were likely associated with haplotype specific regions.

```
# PacBio reads were aligned to the polished genome with minimap2
for i in PacBio_CELL_1.fasta PacBio_CELL_2.fasta ; do minimap2 -I 40G -t 32 -xmap-pb <GENOME> $i | gzip -c - > ${i/.fasta/.paf.gz} ; done
pbcstat *.paf.gz
calcuts PB.stat > cutoffs 2>calcults.log

# de-duplicaation
purge_dups -2 -T cutoffs -c PB.base.cov ./split/Pcro.split.self.paf.gz > dups.bed 2> purge_dups.log
get_seqs -e dups.bed <GENOME>
```



## REPEAT MASKING

Repeats were masked using a custom generated library and the Dfam3.0 library.

```
SEQFILE=purged_genome.fa
DATABASE=purgedDB

BuildDatabase \
  -engine rmblast \
  -name $DATABASE $SEQFILE \
  
RepeatModeler \
  -database $DATABASE \
  -engine rmblast \
  -pa 44 \
  -LTRStruct \
  -genomeSampleSizeMax 500000000

RepeatMasker \
  -dir . \
  -engine ncbi \
  -pa 44 \
  -lib purgedDB-families.fa \
  -gff \
  -html \
  -xsmall \
  $SEQFILE \
```

## ANNOTATION

Annotation was performed with BRAKER2, an automated AUGUSTUS trainer, using RNAseq data as evidence.

### _Mapping RNAseq data_

RNAseq reads were first mapped to the genome using STAR with the 2-Pass method.

```
#   INDEX
STAR \
--runThreadN 32 \
--runMode genomeGenerate \
--genomeDir ${GENOMEDIR} \
--genomeFastaFiles ${GENOME} \

######## 1st PASS
# mapping PE
STAR \
--runThreadN 32 \
--genomeDir ${GENOMEDIR} \
--outFileNamePrefix ${MAPPINGDIR}/1-pass-PE_ \
--outSAMtype BAM Unsorted \
--readFilesCommand zcat \
--readFilesIn SRR1801815_1_paired.fq.gz SRR1801815_2_paired.fq.gz \

# mapping SE
STAR \
--runThreadN 32 \
--genomeDir ${GENOMEDIR} \
--outFileNamePrefix ${MAPPINGDIR}/1-pass_SE_ \
--outSAMtype BAM Unsorted \
--readFilesCommand zcat \
--readFilesIn SRR1801812.fq.gz \

######## 2nd PASS
# mapping PE
STAR \
--runThreadN 32 \
--genomeDir ${GENOMEDIR} \
--outFileNamePrefix ${MAPPINGDIR}/2-pass-PE_ \
--outSAMtype BAM Unsorted \
--readFilesCommand zcat \
--readFilesIn SRR1801815_1_paired.fq.gz SRR1801815_2_paired.fq.gz \
--sjdbFileChrStartEnd ${MAPPINGDIR}/1-pass-PE_SJ.out.tab ${MAPPINGDIR}/1-pass_SE_SJ.out.tab \

# mapping SE
STAR \
--runThreadN 32 \
--genomeDir ${GENOMEDIR} \
--outFileNamePrefix ${MAPPINGDIR}/2-pass_SE_ \
--outSAMtype BAM Unsorted \
--readFilesCommand zcat \
--readFilesIn SRR1801812.fq.gz \
--sjdbFileChrStartEnd ${MAPPINGDIR}/1-pass-PE_SJ.out.tab ${MAPPINGDIR}/1-pass_SE_SJ.out.tab \

########   SAMTOOLS SORTING
samtools sort -@ 31 -o ${MAPPINGDIR}/2-pass-PE_Aligned_sorted.out.bam ${MAPPINGDIR}/2-pass-PE_Aligned.out.bam
samtools sort -@ 31 -o ${MAPPINGDIR}/2-pass_SE_Aligned_sorted.out.bam ${MAPPINGDIR}/2-pass_SE_Aligned.out.bam
```

### _Annotation_

BRAKER2 was then run using these mapped reads

```
braker.pl \
  --workingdir=${OUTPUT} \
  --species=${SPECIES} \
  --softmasking \
  --cleanup \
  --cores=38 \
  --rounds 10 \
  --genome=${GENOME} \
  --bam=${MAPPINGDIR}/2-pass-PE_Aligned_sorted.out.bam,${MAPPINGDIR}/2-pass_SE_Aligned_sorted.out.bam \
  --UTR=on \
  --crf \
  --gff3 \
```

