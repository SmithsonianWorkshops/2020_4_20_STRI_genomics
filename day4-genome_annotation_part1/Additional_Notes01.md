
# Additional Notes 1:
<!-- TOC depthFrom:2 -->

* [Filtering out small scaffolds (optional)](#Filter-out-small-scaffolds-(optional))
* [Checking for contamination using Kraken](#Checking-for-contamination-using-Kraken)

<!-- /TOC -->

### Filter out small scaffolds (optional)

It's common for assemblies to have very short scaffolds ( <500 bp or 1000 bp) that won't be useful for annotation. This is not our case, since the shortest scaffold we have is 3135 bp. But if you are working on your own assemblies and you see that the shortest scaffold is smaller than 500 bp, you can filter those small scaffolds out. It will speed up your genome annotation process, and in my experience, did not affect the genome size and other stats.

To do so, we will use `bioawk`, which is available as module. This command also can be run on the interactive queue.



```
module load bioinformatics/bioawk
bioawk -c fastx '{ if(length($seq) > 499) { print ">"$name; print $seq }}' input.fasta > filtered.fasta
```

**Observation**: this bioawk command will remove any scaffolds of the specified size and below. Since I want to keep the 500bp scaffolds, I used 499 instead of 500.  


### Checking for contamination using Kraken
We ran Kraken on the full assembly and used the `--confidence` flag. One important thing to keep in mind is that here, we are 

##### Job file
- Queue: short
- PE: multithread 10 (-pe mthread)
- Memory: 4G
- Module: `module load bioinformatics/kraken`
- Commands:

`kraken2 --db /data/genomics/db/Kraken/kraken2_db/ --report kraken_report --use-names --confidence --threads $NSLOTS ASSEMBLY.fa`


**Observation: here, we are keeping the log and error files separate. The log file will have the actual output that we want.


**Output files:**

- kraken2.err
- kraken_report
- kraken2.log


Since the full kraken2 output was saved in the log file, first we will get only the list of contigs:

`grep "Contig" kraken2.log > kraken2_results.txt`


##### Get list of human and unclassified contigs:
By doing this, we are removing anything classified as bacteria, plasmids or viruses. We keep the unclassified because they might contain real contigs. 

`grep "unclassified\|Homo sapiens" kraken2_results.txt | cut -f2 > assembly.fa.HsU.list`

    
##### Using `samtools`, we will extract from the assembly only the contigs identified as Human or unclassified. Bacteria, viruses and plasmids will be excluded.

```
module load bioinformatics/samtools
xargs samtools original_assembly.fa < assembly.fa.HsU.list > new_assembly.fa
```

##### We applied this to a prairie dog genome (check the publication [here](https://academic.oup.com/gbe/advance-article/doi/10.1093/gbe/evaa069/5819143?searchresult=1)) 
The genome size decreased around 1.84%, but all the other stats (with exception of the longest and shortest scaffold/contig) were improved. This assembly was submitted to Genbank and accepted with no errors.
