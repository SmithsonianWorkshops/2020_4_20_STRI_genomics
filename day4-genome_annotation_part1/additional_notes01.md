Additional Notes 1:
Filtering out small scaffolds (optional)
Checking for contamination using Kraken
Filter out small scaffolds (optional)
It’s common for assemblies to have very short scaffolds ( <500 bp or 1000 bp) that won’t be useful for annotation. This is not our case, since the shortest scaffold we have is 3135 bp. But if you are working on your own assemblies and you see that the shortest scaffold is smaller than 500 bp, you can filter those small scaffolds out. It will speed up your genome annotation process, and in my experience, did not affect the genome size and other stats.

To do so, we will use bioawk, which is available as module. This command also can be run on the interactive queue.

module load bioinformatics/bioawk
bioawk -c fastx '{ if(length($seq) > 499) { print ">"$name; print $seq }}' input.fasta > filtered.fasta
Observation: this bioawk command will remove any scaffolds of the specified size and below. Since I want to keep the 500bp scaffolds, I used 499 instead of 500.

Checking for contamination using Kraken
We ran Kraken on the full assembly and used the --confidence flag. One important thing to keep in mind is that here, we are

Job file
Queue: short
PE: multithread 10 (-pe mthread)
Memory: 4G
Module: module load bioinformatics/kraken
Commands:
kraken2 --db /data/genomics/db/Kraken/kraken2_db/ --report kraken_report --use-names --confidence --threads $NSLOTS ASSEMBLY.fa

**Observation: here, we are keeping the log and error files separate. The log file will have the actual output that we want.

Output files:

kraken2.err
kraken_report
kraken2.log
Since the full kraken2 output was saved in the log file, first we will get only the list of contigs:

grep "Contig" kraken2.log > kraken2_results.txt

Get list of human and unclassified contigs:
By doing this, we are removing anything classified as bacteria, plasmids or viruses. We keep the unclassified because they might contain real contigs.

grep "unclassified\|Homo sapiens" kraken2_results.txt | cut -f2 > assembly.fa.HsU.list

Using samtools, we will extract from the assembly only the conti .
grep "^C" kraken2_pdog_assembly_notHsapiens.txt | cut -f2 > kraken2_pdog_assembly_notHsapiens_contigs.txt

We will remove the contigs in this list from the original assembly. We had a total of 1308 contigs that were classified as not human - which means that they corresponded to bacteria, plasmids or viruses.

The new assembly stats are:
pilon_pdog_minus1308.fasta	Contig statistics	Scaffold statistics
Total bp	2669691975	2674371627
Total number	15346	12628
N50	686670	824613
L50	1096	910
GC content	40.02%	40.02%
Median size	35012	43162
Mean size	173966.63	211781.09
Longest	4996552	5700915
Shortest	1454	1454
The genome size decreased around 1.84%, but all the other stats (with exception of the longest and shortest scaffold/contig) were improved. This assembly was submitted to Genbank and accepted with no errors.

Select a repo
