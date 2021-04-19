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

Polishing with NextPolish requires file of file names for long and short reads, and a config file. The ```run.cgf``` file was formatted as below, with .

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


