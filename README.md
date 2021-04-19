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
get_seqs dups.bed <GENOME>
```
