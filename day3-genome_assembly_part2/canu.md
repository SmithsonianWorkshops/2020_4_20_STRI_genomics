## 1. Running Canu

### What is Canu?

Canu either PacBio or Oxford Nanopore Technologies. https://canu.readthedocs.io/en/latest/quick-start.html
From the Canu docs: Canu specializes in assembling PacBio or Oxford Nanopore sequences. Canu operates in three phases: correction, trimming and assembly. The correction phase will improve the accuracy of bases in reads. The trimming phase will trim reads to the portion that appears to be high-quality sequence, removing suspicious regions such as remaining SMRTbell adapter. The assembly phase will order the reads into contigs, generate consensus sequences and create graphs of alternate paths.

For eukaryotic genomes, coverage more than 20x is enough to outperform current hybrid methods, however, between 30x and 60x coverage is the recommended minimum. More coverage will let Canu use longer reads for assembly, which will result in better assemblies.

* Set up a Canu job:
	+ Module: ```bioinformatics/canu```
	+ Commands: ```canu useGrid=false -d <directory-for-output> -p <assembly-prefix> genomeSize=2.5g -pacbio-raw <reads-file>.fastq maxThreads=$NSLOTS```
	+ Try this on some mammal data here: ```/data/genomics/workshops/STRI_genomics/mammal_pacbio.fasta``` 
    + Hint: this job needs to run on the high memory queue. 
