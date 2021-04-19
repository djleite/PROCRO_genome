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

ASSEMBLY
========

Flye was used for assembly of the PacBio reads.

```flye \
  --pacbio-raw ${READ1} ${READ2} \
  -g 2.53g \
  -o ${GENOME_OUT} \
  -t 40 \
  -i 1 \
  --asm-coverage 75 \
  -m 8000 \```


