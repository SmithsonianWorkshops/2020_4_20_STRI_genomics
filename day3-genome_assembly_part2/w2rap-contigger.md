## 1. Running w2rap-contigger

### What is w2rap-contigger?


An Illumina PE genome contig assembler, can handle large (17Gbp) complex (hexaploid) genomes. http://bioinfologics.github.io/the-w2rap-contigger/


* Set up a w2rap-contigger job:
	+ Create a directory in which to run the job.
	+ Module: ```bioinformatics/w2rap-contigger```
	+ Commands: ```w2rap-contigger -o <output-directory> -p example -r <read1>.fastq,<read2>.fastq -K 200```
	+ Try this on some the Red Siskin Illumina data here: ```/data/genomics/dikowr/SMSC/Illumina_all/*.fq``` 
    + Hint: this job needs to run on the high memory queue. 
