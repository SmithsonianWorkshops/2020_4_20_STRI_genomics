## 1. Running Read Bean (Wtdbg2) assembler

### What is Wtdbg?

Wtdbg2 is a sequence assembler for long noisy reads produced by either PacBio or Oxford Nanopore Technologies. This program cuts reads into 1024bp segments, merges similar segments into a vertex and connects vertices based on the segment adjacency on reads. These sequence of steps procudes a graph that the authors called fuzzy Bruijn graph (FBG), which is similar to the Bruijn graph but allows mismatches and keeps read paths when collapsing k-mers . The program uses two steps. The first step is the assembler and the second step is the consenser.[(see Wtdbg2)](https://github.com/ruanjue/wtdbg2).

* You can run Wtdb2 in two ways: 
1 by running a perl script that calls the two main commands (see Wtdbg2 github) or 
2 by running the individually commands

* here you will do it step by step.

* First step, create a job file for the first command:
	+ Module: ```bioinformatics/wtdbg2```
	+ Commands: ```wtdbg2 -g 4.6m -t $NSLOTS -x rs -i /path/to/reads_file.fa_or_fq -fo prefix_for_your_output_file```
	+ ```-g``` = estimated genome size in megabases (m) of gigabases (g)(i.e, 4.6m)  
	+ ```-i``` = input file fasta or fastq  
	+ ```-x``` = preset parameters depending on the sequencing technology, check ```./wtdbg2 -help ``` for other parameters. In this case this  ```-x rs``` is for Pacbio rs sequencing plataform.
	+ Hint: this job needs to run on the high memory queue. 
	
	
* Second step, create a job file for the second command:
	+ Module: ```bioinformatics/wtdbg2```
	+ Commands: ```wtpoa-cns -t $NSLOTS -i prefix.ctg.lay.gz -fo genome_draft_raw.fa```
	+ ```-t``` = # of threads
	+ ```-i``` = prefix.ctg.lay.gz (this is the one of the results file from previous command)
	+ ```-x``` = name for your draft genome in fasta format
	+ Hint1: this job needs to run on the high memory queue.
	+ Hint2: check your assembly with assembly-stats

* You can used long read data and short read data to polish your draft genome by using these scripts:

 + For long reads, you can used minimap2 to map your long reads to your draft genome, after that you will need to sort and filtered by reads that have the higher match to your draft genome, finally you will run de concensus command.
 
   + ```minimap2 -t $NSLOTS -ax map-pb -r2k dbg.raw.fa reads.fa.gz | samtools sort -@4 >dbg.bam```
   + ```samtools view -F0x900 dbg.bam | ./wtpoa-cns -t $NSLOTS -d dbg.raw.fa -i - -fo dbg.cns.fa```
   + Hint1: you need to load modules for minimap and samtools
   + Hint2: check your assembly with assembly-stats

 + For short reads, you can used bwa to map your short reads to your draft genome, after that you will sort the mapped file and finanlly run the concesus command.
 
   + ```bwa index dbg.cns.fa```
   + ```bwa mem -t $NSLOTS dbg.cns.fa sr.1.fa sr.2.fa | samtools sort -O SAM | ./wtpoa-cns -t $NSLOTS -x sam-sr -d dbg.cns.fa -i - -fo dbg.srp.fa```
   + Hint1: you need to load modules for bwa and samtools
   + Hint2: check your assembly with assembly-stats
