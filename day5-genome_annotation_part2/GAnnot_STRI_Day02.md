
# Genome annotation - STRI 2020 (Day 2)
<!-- TOC depthFrom:2 -->

 * [AUGUSTUS](#AUGUSTUS)
     * [Partition the assembly into scaffolds](#Partition-the-assembly-into-scaffolds)
 	* [A brief introduction to LOOPS](#A-brief-introduction-to-LOOPS)
 	* [Combining the results](#Combining-the-results)
 * [Running Blast](#Running-Blast)
     * [Split fasta into sub-files with 100 sequences](#Split-fasta-into-sub-files-with-100-sequences)
     * [Create blast job array](#Create-blast-job-array)
     * [Merge the xml files](#Merge-the-xml-files)
 * [Running Blast2GO](#Running-Blast2GO)

<!-- /TOC -->

### AUGUSTUS


For the AUGUSTUS job, we need the following input files:

1. assembly (masked)
2. merged RM and E hints file
3. extrinsic file
4. retraining parameters (from BUSCO)


AUGUSTUS will run serially, one scaffold at a time. In order to speed up the process, we can break the assembly into scaffolds and process them in paralel. To do so, we will use a script from EVM (EVidenceModeller) to split the assembly and the hints file, and create job arrays for AUGUSTUS.

#### Partition the assembly into scaffolds 

EVM (EVidence Modeller, Haas et al. 2008) is a program that combines *ab initio* gene predictions and protein and transcript alignments into weighted consensus gene structures. We will use an EVM script that splits the assembly into folders, with one scaffold per folder plus its corresponding hints file (in gff).

We don't have EVM installed as a module on Hydra, but you can download ([https://github.com/EVidenceModeler/EVidenceModeler/archive/v1.1.1.tar.gz](https://github.com/EVidenceModeler/EVidenceModeler/archive/v1.1.1.tar.gz)) and extract it in your `augustus` folder. The script is in the folder EVmutils. This script runs fast, so we will use the interactive queue to run it. 

<details><summary>HINT</summary>
<p>

---

**From your `augustus` folder**  

```
wget https://github.com/EVidenceModeler/EVidenceModeler/archive/v1.1.1.tar.gz
tar -zxf v1.1.1.tar.gz

```
---


</p>
</details>


Now `cd` to `scaffolds` and run the following command from it:

```
module load bioinformatics/bioperl

perl ../EVidenceModeler-1.1.1/EvmUtils/partition_EVM_inputs.pl \
     --genome ../../repmasker/siskin_10largest.fasta.masked \
     --gene_predictions ../hints/siskin_hints_RM_E.gff3 \
     --segmentSize 10000000 --overlapSize 3000000 \
     --partition_listing partitions_list.out
```

Now you should have 10 folders, each one with one scaffold and its corresponding hints file. They all retained the same name or the original file, and the folders are identified as `Contig1977_pilon`, `Contig20_pilon`, etc.


#### A brief introduction to LOOPS

We will take a "break" from this pipeline to talk about loops. 
Loops allow you to execute repetitive tasks multiple times with a single command. For example:

Imagine that I have a folder with the following files: 

```
01.txt	02.txt	03.txt	04.txt	01.dat	02.dat
```

And let's say that I want to make a copy of all files with the extension `.txt`
and add the word backup in from of it. What are the options? 

1. I can type each command individually

	```
	cp 01.txt backup_01.txt
	cp 02.txt backup_02.txt
	cp 03.txt backup_03.txt
	cp 04.txt backup_04.txt
	```
	
2. Or I can write a loop, that will iterate over all `txt` files:

	```
	for f in *.txt; do
		cp ${f} backup_${f};
	done
	```

**What does this loop mean?**

- `for`: starts a for loop  
- `f in *.txt`: this statement takes all files that have the extension `.txt` and assign them to the variable of name `f`. You can call your variable anything you want (really, anything). The for loop will iterate over all files, one at a time.
- `do`: before the command, you need to add the word `do`.
- `cp ${f} backup_${f}`: command to be executed. Here we are copying all `txt` files and adding the preffix backup_.
- `done`: closes the loop.

**Why is it important?**

For loops are very useful when you have multiple files to process. We will use a for loop to submit our augustus jobs. 


Now, let's create our augustus job file.

#### Job file: augustus.job
- Queue: short
- PE: serial
- Memory: 2G
- Module: `module load bioinformatics/augustus/3.3`
- Commands:
	
```
export AUGUSTUS_CONFIG_PATH="/pool/genomics/username/smsm_2019/genome_annot/augustus/config"
#
augustus --strand=both --singlestrand=true \
--hintsfile=${1}/siskin_hints_RM_E.gff3 \
--extrinsicCfgFile=extrinsic.M.RM.E.cfg \
--alternatives-from-evidence=true \
--gff3=on \
--uniqueGeneId=true \
--softmasking=1 \
--species=BUSCO_siskin_2188842729 \
${1}/siskin_10largest.fasta.masked > ../output/siskin_augustus_${1}.gff
#
echo = `date` job $JOB_NAME done
#
```

Before we submit the job, let's exit from the interactive queue back to the login node by typing `exit`
Now, let's make sure we are in the correct folder. We will submit this job from the folder `scaffolds` (the one that has all the 10 folders).

To submit the job, use the following command (a for loop)

```
for dir in Contig*/; do 
 out=${dir/\/}; qsub -N augustus_${out} -o ../../logs/augustus_${out}.log ../../jobs/augustus.job ${out};
done
```


##### Understanding the commands:

- `for dir in scaffold_*/;`: This loop will iterate over all folders that correspond to the pattern (in this case, name starts with scaffold-)
- `do out=${dir/\/};`: this command will create a new variable called `out` (you can give any name you want). This new variable will be based on the variable `$dir`, without the trailing slash "/".
- `qsub -N test_${out} -o ../../logs/augustus_${out}.log ../../jobs/augustus.job ${out}`. Here we will finally submit the job, and some of the job parameters will be overwritten:
	- `-N`: job name. We will give a name that includes the variable that contains the scaffold name.
	- `-o`: log file. Same as the job name, The goal here is to one log file per scaffold.
	- `../../jobs/augustus.job ${out};`: in this part, we are calling on the job file `jobs/augustus.job` to be submitted. We need to include the variable `${out}` for the job to run. In the job file, we did not provide any specific paths (remember that we used the variable `${1}`?). The variable `${1}` in the job corresponds to the variable that comes after the job file (in this case, `${out}`).
- `done`: closes the loop.
- `--hintsfile=${1}/hints_RM.E.gff3`: The variable `${1}` is used to identify each folder.
	-  The same applies to `${1}/siskin_10largest.fasta.softmasked` and the output file `siskin_augustus_${1}.gff`.


The job is submitted from the directory where all the `Contig*` folders are located, and not from inside each folder.
Also, I'm saving all output files in a separate directory `output` to facilitate post-processing.


#### Combining the results

Use the script `join_aug_pred.pl` from AUGUSTUS (use the interactive queue `qrsh`, load the module `augustus` and run the commands from the `output` folder).

1. Concatenate all output files from augustus in numerical order:
`cat $(find . -name "siskin_augustus_*.gff" | sort -V) > siskin_augustus.concat`

2. Join the results using the `join_aug_pred.pl`
`cat siskin_augustus.concat < /share/apps/bioinformatics/augustus/conda/3.3.2/bin/join_aug_pred.pl >> siskin_augustus_all.gff`

3. Extract each gene model as protein (aa) and nucleotide, as well as each CDS individually:

	`getAnnoFasta.pl siskin_augustus_all.gff --seqfile=../../repmasker/siskin_10largest.fasta.masked --chop_cds --protein=on --codingseq=on`
	
	As a result, we have three new files:
	
    ```
    siskin_augustus_all.cdsexons
    siskin_augustus_all.codingseq
    siskin_augustus_all.aa
    ```

4. Let's explore those files

	<details><summary>HINT</summary>
	<p>
	
	---
	
	```
	head siskin_augustus_all.[ac]*
	
	for f in siskin_augustus_all.[ac]*; 
		do 
		echo $f; 
		grep -c ">" $f;  
		done
	```
	---
	
	</p>
	</details>


### Running Blast
After running AUGUSTUS, we will run BLAST to try to characterize the gene models (putative genes, coding regions) identified by AUGUSTUS. We will use the file with aminoacid sequences `siskin_augustus_all.aa`

##### Folder structure:

First, we will create the folders fa, input_files, jobs, logs, results and xml inside the blast folder.

	blast
		|_ fa			split fasta files
		|_ input_files 	input files before starting the analysis
		|_ jobs			job files
		|_ logs			log files
		|_ results		FINAL results
		|_ xml			xml outputs


<details><summary>HINT</summary>
<p>
	
---
	
```
mkdir fa input_files jobs logs results xml
```
---
	
</p>
</details>


Nos, let's copy the file `siskin_augustus_all.aa` from the folder `augustus/output` to the folder `blast/input_files`.


<details><summary>HINT</summary>
<p>
	
---
	
`cd` into the folder `input_files`, and then:
	
```
cp ../../augustus/output/siskin_augustus_all.aa .
```
---
	
</p>
</details>


#### Split fasta into sub-files with 100 sequences
To make BLAST more efficient, we will split the amioacid fasta file into multiple files with 50 sequences each. 


**Generic command**
    
	awk 'BEGIN {n_seq=0;} /^>/ \
	{if(n_seq%50==0){file=sprintf("prefix_%d.fa",n_seq);} \
	print >> file; n_seq++; next;} { print >> file; }' < augustus_CDS.fasta
	
(Source: https://www.biostars.org/p/13270/)

**Our command**

	awk 'BEGIN {n_seq=0;} /^>/ {if(n_seq%50==0){file=sprintf("../fa/siskin_augustus_aa_%d.fa",n_seq);} print >> file; n_seq++; next;} { print >> file; }' < siskin_augustus_all.aa


Some notes about this script: 

- On the second line, the number correspond to the number of sequences in each file (100).
- The prefix is the name of your new files, followed by their number ID.


#### Create blast job array

Now, we will create a blast job with tasks, or a job array. In very non-technical terms, think about a job array as an item on a to-do list that has multiple sub-tasks. To run this job array, we will create a very different kind of job, with three separate files:  
	* list  
	* job header  
	* list of commands  

	
Let's work from the `blast/jobs` folder

##### List
This list contains all the files that will be used as input for the job array. To generate this list, we will save the output of the command `ls` to a file called `blast.list` in the folder `jobs`

From the folder `blast/fa`, do:

```
ls *.fa > ../jobs/blast.list
```

##### Job header
This file contains all the information to run the job that is normally found in the header of a job file. You can copy a template from `data/genomics/workshops/STRI2020/blastp_template.conf`. Let's explore the file:

```
qsub \
-q mThC.q \
-l mres=6G,h_data=6G,h_vmem=6G \
-cwd -j y -N blastp \
-t 1-50 \
-o '../logs/blastp_$TASK_ID.log' \
-b y $PWD/blastp.sh
```

How is it different from a regular job file? 

Parameter to change:  
	`-t 1-50` should be changed to the maximum number of files in the folder `blast/fa` (which is the same as the number of lines in the file `blast.list`)

##### List of commands
This file contains the job commands, as well as the modules to be loaded. You can copy the template `blastp_template.sh` from  `data/genomics/workshops/STRI2020/`

```
#!/bin/sh
#
# ----------------Modules------------------------- #
source /etc/profile.d/modules.sh
module load bioinformatics/blast
#
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
#
i=$SGE_TASK_ID
P=`awk "NR==$i" my.list`
#
echo $P
#
blastp -query ../fa/${P} -db nr -outfmt 5 -max_target_seqs 10 -evalue 1e-4 -out ../xml/${P}.xml
#
echo = `date` job $JOB_NAME done
#
```

Let's change the name of the list: currently `my.list`. Use the name of the list you created.

#### Running the job:
To run this job, you will use a different command. 

`source blastp_siskin.conf`

You should see a message that says `Your job-array 1012884.1-38:1 ("blastp_") has been submitted`. 

Use `qstat` to see your jobs running. And also explore the folders `logs` and `xml`.

#### Merge the xml files

After all the jobs are done, you need to merge all xml files into one single file. You can use `cat` for that:
From the `xml` folder, do:

	cat $(find . -name "*.xml" | sort -V) > ../results/siskin_blastp_all.xml


### Running Blast2GO

We used Blast2GO to funcionally annotate the genome. First, we ran Blast (v2.6.0+) only on the gene models (either blastx on siskin_augustus_all.codingseq or blastp on siskin_augustus_all.aa) identified by AUGUSTUS. 

This [page](https://confluence.si.edu/display/HPC/Running+BLAST2GO+on+Hydra) on the Hydra Wiki has more info about how to run blast2go on Hydra.

From your blast2go folder: 

1. Create your blast2go job file:

```
# /bin/sh
# ----------------Parameters---------------------- #
#$ -S /bin/sh
#$ -q lTb2g.q
#$ -l b2g,mres=24G,h_data=24G,h_vmem=24G
#$ -cwd 
#$ -j y 
#$ -N b2g-siskin 
#$ -o b2g-siskin.log
#
# ----------------Modules------------------------- #
#
module load bioinformatics/blast2go
#
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
#
#
 
# Make local copy of cli.prop
# (this overwrites any existing cli.prop file in the current directory and only needs to be done once)
hydracliprop
 
# Run example dataset:
runblast2go \
  -properties cli.prop \
  -loadfasta ../blast/input_files/siskin_augustus_all.aa \
  -loadblast ../blast/results/siskin_blastp_all.xml \
  -mapping \
  -annotation \
  -nameprefix siskin \
  -saveb2g -saveannot -savereport -savelorf -saveseqtable
  -statistics all
  -exportgeneric
 
#
echo = `date` job $JOB_NAME done


```

Blast2go runs on its own special queue on Hydra, and only one job can be run at a time. 

After the job is done, you will have several output files: 
- pdog.b2g: blast2go project file. It can be imported to the GUI version of blast2go. 
- pdog.annot: contains all the possible GO terms for each sequence.
- pdog.txt: tabular version of the blast2go project
- pdog.fasta: fasta sequences
- pdog.pdf: report, with summary stats and graphs. 

The command line version of b2go is a very practical and efficient way of running the program, but has less resources than the GUI option. If you have access to the b2go GUI, you can run other analyses, such as interproscan, 
