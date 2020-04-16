
## Summary of steps:
<!-- TOC depthFrom:2 -->
 * [1. Folder structure](#1-folder-structure)
 * [2. Get the assembly stats with assembly-stats](#2-Get-the-assembly-stats-with-assembly-stats)
 * [3. Filter out small scaffolds](#3-Filter-out-small-scaffolds-(optional))
 * [4. Run BUSCO using the --long flag](#4-Run-BUSCO-using-the---long-flag)
 	* [4a. Using BUSCO output for the AUGUSTUS run](#4a-Using-BUSCO-output-for-the-AUGUSTUS-run)
 * [5. Masking and annotating repetitive elements](#5-Masking-and-annotating-repetitive-elements)
 * [6. Run BLAT](#6-Run-BLAT)
 * [7. Create Augustus hints](#7-Create-Augustus-hints)
 	* [Creating hint files from RepeatMasker](#Creating-hint-files-from-RepeatMasker)
	* [Creating hint files from BLAT](#Creating-hint-files-from-BLAT)
	* [Merging hints files](#Merging-hints-files)
 * [8. Edit the extrinsic file to add only the evidence you have](#8-Edit-the-extrinsic-file-to-add-only-the-evidence-you-have)
 * [9. AUGUSTUS](#9-AUGUSTUS)
 	* [9a. A brief introduction to LOOPS](#9a-A-brief-introduction-to-LOOPS)
 * [Running Blast and Blast2GO](#Running-Blast-and-Blast2GO)
 * [10. Split fasta into sub-files with 100 sequences](#10-Split-fasta-into-sub-files-with-100-sequences)
 * [11. Create blast job array](#11-Create-blast-job-array)
 * [12. Merge the xml files](#12-Merge-the-xml-files)
 * [13. Running Blast2GO](##13-Running-Blast2GO)
 * [14. Create a genome browser with JBrowse](#14-Create-a-genome-browser-with-Jbrowse)

<!-- /TOC -->

## Morning - part 1

### 1. Folder structure

Now let's create our folder structure for this workshop. It's easier to troubleshoot any issues if we are all working within the same framework. First, we will create a folder called `genome_annot` in your `/pool/genomics/username` folder. 

```
cd /pool/genomics/your_username/scmc_2019

mkdir genome_annot

```

Next, we will change directories to the genome_annot folder and we will create a several folders that we will use today and tomorrow. Here's the list of folders:

- assembly
- busco
- repmasker
- b2go
- blast
- blat
- augustus
- jbrowse
- logs
- jobs



<details><summary>SOLUTION</summary>
<p>

```
mkdir assembly busco repeatmasker b2go blast blat augustus jbrowse logs jobs

```
If you type the command `tree`, this is what you should see:

```
.  
|__ assembly  
|__ busco  
|__ repmasker  
|__ b2go 
|__ blast  
|__ blat  
|__ augustus  
|__ jbrowse  
|__ jobs  
|__ logs  

```

</p>
</details>


If we follow this folder structure, we will have all jobs in the same folder, as well as all log files. Also, the results of each job will be organized by software, which will facilitate finding everything later. 

**Submitting jobs**: With this folder structure, we will save all the job files in the `jobs` folder, and they will be submitted from there.   
***Exception:** BUSCO doesn't allow you to provide a path for the output files, so you should run the job from the busco folder, like this: `qsub ../jobs/busco.job`*


### 2. Get the assembly stats with `assembly-stats`

For this workshop, we will use a file with the 10 largest scaffolds of the *Spinus cucullatusi* (Red Siskin) assembly. You can copy the assembly file from `/data/genomics/workshops/SMSC_2019/siskin_10largest.fasta`. You should copy it to your `assembly` folder.

<details><summary>SOLUTION</summary>
<p>

From the `assembly` folder:  
`cp /data/genomics/workshops/SMSC_2019/siskin_10largest.fasta .`

</p>
</details>


The first step will be to get some basic stats about this assembly. We will use the module `assembly_stats` for that, we will run it from the interactive queue (**we never run anything from the login node**).  

To access the interactive queue, type `qrsh`. Now you are logged to a compute node.


---
**Important**   
If you type `pwd`, you will see that you are back to your home directory `/home/username/`. You need to `cd` to the `assembly` folder in the `genome_annot`. 

<details><summary>SOLUTION</summary>
<p>

`cd /pool/genomics/username/scmc_2019/genome_annot/assembly`

</p>
</details>

---

Now we will load the `assembly_stats` module, and run the `assembly_stats` script. 


	module load bioinformatics/assembly_stats
	assembly_stats siskin_10largest.fasta

Your results should look like this:

```
{
  "Contig Stats": {
    "L10": 0,
    "L20": 1,
    "L30": 3,
    "L40": 4,
    "L50": 5,
    "N10": 4781185,
    "N20": 3790167,
    "N30": 3728836,
    "N40": 3578115,
    "N50": 3464414,
    "gc_content": 42.05073278445502,
    "longest": 4781185,
    "mean": 1594594.7307692308,
    "median": 966694.0,
    "sequence_count": 26,
    "shortest": 8485,
    "total_bps": 41459463
  },
  "Scaffold Stats": {
    "L10": 0,
    "L20": 1,
    "L30": 2,
    "L40": 3,
    "L50": 4,
    "N10": 5638391,
    "N20": 5116614,
    "N30": 4637440,
    "N40": 4537267,
    "N50": 3769381,
    "gc_content": 42.05073278445502,
    "longest": 5638391,
    "mean": 4146329.1,
    "median": 3749108.5,
    "sequence_count": 10,
    "shortest": 3391225,
    "total_bps": 41463291
  }
}
```

Alternatively, you can save your results to a file, instead of having them printed to the screen. To do so, you use the following command:

	assembly_stats siskin_10largest.fasta > siskin_10largest.stats
	
If you use the command`cat`, you will see that the output file has the same results:

	cat siskin_10largest.stats

---
**Challenge**   
Run the assembly stats on the full assembly. Compare the results betweeen the complete assembly and the one contig.

<details><summary>SOLUTION</summary>
<p>

```
{
  "Contig Stats": {
    "L10": 41,
    "L20": 103,
    "L30": 188,
    "L40": 297,
    "L50": 437,
    "N10": 2198517,
    "N20": 1512377,
    "N30": 1167155,
    "N40": 893946,
    "N50": 699255,
    "gc_content": 42.76010455605358,
    "longest": 4781185,
    "mean": 147389.64487843763,
    "median": 23620.0,
    "sequence_count": 7527,
    "shortest": 1231,
    "total_bps": 1109401857
  },
  "Scaffold Stats": {
    "L10": 35,
    "L20": 87,
    "L30": 156,
    "L40": 248,
    "L50": 365,
    "N10": 2454989,
    "N20": 1888260,
    "N30": 1413230,
    "N40": 1077080,
    "N50": 841639,
    "gc_content": 42.76010455605358,
    "longest": 5638391,
    "mean": 200011.9994611101,
    "median": 35502.0,
    "sequence_count": 5567,
    "shortest": 2132,
    "total_bps": 1113466801
  }
}
```

</p>
</details>

---


**Important: From now on, we will run all the other steps as jobs. To exit back to the login node, type** `exit`


### 3. Filter out small scaffolds (optional)

It's common for assemblies to have very short scaffolds ( <500 bp or 1000 bp) that won't be useful for annotation. This is not our case, since the shortest scaffold we have is 3135 bp. But if you are working on your own assemblies and you see that the shortest scaffold is smaller than 500 bp, you can filter those small scaffolds out. It will speed up your genome annotation process, and in my experience, did not affect the genome size and other stats.

To do so, we will use `bioawk`, which is available as module. This command also can be run on the interactive queue.



```
module load bioinformatics/bioawk
bioawk -c fastx '{ if(length($seq) > 499) { print ">"$name; print $seq }}' input.fasta > filtered.fasta
```

**Observation**: this bioawk command will remove any scaffolds of the specified size an below. Since I want to keep the 500bp scaffolds, I used 499 instead of 500.  
  

### 4. Run BUSCO using the --long flag

BUSCO (Simão et al. 2015; Waterhouse et al. 2017) assesses completeness by searching the genome for a selected set of single copy orthologous genes. There are several databases that can be used with BUSCO and they can be downloaded from here: [https://buscos.ezlab.org](https://buscos.ezlab.org). 

---
##### *** Before running BUSCO, copy the file augustus/config folder to a place where you have writing privileges***
The AUGUSTUS config folder can be found here:  `/share/apps/bioinformatics/augustus/conda/3.3.2/config/`. Copy it to the folder `augustus` inside your `genome_annot` folder.

<details><summary>SOLUTION</summary>
<p>
Assuming you are in the folder `/pool/genomics/username/scmc_2019/genome_annot/augustus`:  

`cp -r /share/apps/bioinformatics/augustus/conda/3.3.2/config/ .`

</p>
</details>

---

**Database:**

For this workshop, we will use the Aves database. Download it to the folder `busco` using the command `wget` and extract it. Let's `cd` to the directory `busco` first.

	wget https://buscos.ezlab.org/datasets/aves_odb9.tar.gz
	tar -zxf aves_odb9.tar.gz


According to the BUSCO manual, the `--long` flag turns on Augustus optimization mode for self-training. It can be used as a training set for AUGUSTUS.


#### Job file 1: busco_contig.job

- PE: multi-thread
- Number of CPUs: 4
- Memory: 6G (6G per CPU, 24G total)
- Module: `module load bioinformatics/busco`
- Commands:

```
export AUGUSTUS_CONFIG_PATH="/pool/genomics/username/smsc_2019/genome_annot/augustus/config"
#
run_BUSCO.py --long -o siskin -i ../assembly/siskin_10largest.fasta -l aves_odb9 -c $NSLOTS -m genome
```

##### Explanation:
```
--long: turn on Augustus optimization mode for self-training.
-o: name of the output folder and files
-i: input file (FASTA)
-l: path to the folder containing the database of BUSCOs (the one you downloaded from the BUSCO website).
-c: number of CPUs
-m: mode (options are genome, transcriptome, proteins

```
---
##### *** IMPORTANT ***

BUSCO doesn't have an option to redirect the output to a different folder. For that reason, we will submit the BUSCO job from the `busco` folder. Assuming you just created the job file in the `jobs` folder:

```
cd ../busco
qsub ../jobs/busco_siskin.job
```
---


##### 4a. Using BUSCO output for the AUGUSTUS run
Copy the folder `run_siskin/augustus_output/retraining_parameters` to your folder `augustus/config/species`. Rename the folder with the name of the run (you can find it by looking at the file prefix inside the folder). In this case, we will rename the folder `BUSCO_siskin_2598564919`

```
BUSCO_siskin_2598564919_exon_probs.pbl
BUSCO_siskin_2598564919_igenic_probs.pbl
BUSCO_siskin_2598564919_intron_probs.pbl
BUSCO_siskin_2598564919_metapars.cfg
BUSCO_siskin_2598564919_metapars.cgp.cfg
BUSCO_siskin_2598564919_metapars.utr.cfg
BUSCO_siskin_2598564919_parameters.cfg
BUSCO_siskin_2598564919_parameters.cfg.orig1
BUSCO_siskin_2598564919_weightmatrix.txt

```

<details><summary>HINT</summary>
<p>

---
**From the busco folder**  

```
cp run_siskin/augustus_output/retraining_parameters ../augustus/config/species/
cd ../augustus/config/species/
ls retraining_parameters
mv retraining_parameters BUSCO_siskin_2598564919
```


---

</p>
</details>


<details><summary>ADDITIONAL INFO</summary>
<p>

---
You can also modify all filenames to match just the species ("siskin"). But in this case, you need to rename all files AND replace the current species id `BUSCO_siskin_2188842729` by "siskin" in all files using *sed*.

In this case, these are the steps we would follow:

- Rename the folder `retraining_parameters` to `siskin`  
	`mv retraining_parameters siskin`
	
- Rename all files in the folder  
	```
	for f in *; do 
		mv $f ${f/BUSCO_siskin_2598564919/siskin};
	done
	```
	
- Replace the names in the files using `sed`  
	`sed -i 's/BUSCO_siskin_2598564919/siskin/g' *`

---

</p>
</details>


### 5. Masking and annotating repetitive elements

From the RepeatMasker manual (Smit, 2013-2015):
> Repeatmasker is a program that screens DNA sequences for interspersed repeats and low complexity DNA sequences.The output of the program is a detailed annotation of the repeats that are present in the query sequence as well as a modified version of the query sequence in which all the annotated repeats have been masked. 


#### Job file: repeatmasker.job
- Queue: medium
- PE: multi-thread
- Number of CPUs: 10
- Memory: 4G
- Module: `module load bioinformatics/repeatmasker`
- Commands:

```
RepeatMasker -species chicken -pa $NSLOTS -xsmall -dir ../repmasker ../assembly/siskin_10largest.fasta

```
##### Explanation:
```
-species: species/taxonomic group repbase database (browse available species here: 
 https://www.girinst.org/repbase/update/browse.php)
-pa: number of cpus
-xsmall: softmasking (instead of hardmasking with N)
-dir ../repmasker: writes the output to the directory repmasker

```

##### Output files:
- siskin_10largest.fasta.tbl: summary information about the repetitive elements
- siskin_10largest.fasta.masked: masked assembly (in our case, softmasked)
- siskin_10largest.fasta.out: detailed information about the repetitive elements, including coordinates, repeat type and size.

##### About the species:
- You can use the script `queryTaxonomyDatabase.pl` from the RepeatMasker module to search for your species of interest. 

	`queryTaxonomyDatabase.pl -species cat`
	
	**Output:**
	
	```
	RepeatMasker Taxonomy Database Utility
	======================================
	Species = cat
	Lineage = Felis catus
	          Felis
	          Felinae
	          Felidae
	          Feliformia
	          Carnivora
	          Laurasiatheria
	          Boreoeutheria
	          Eutheria
	          Theria Mammalia
	          Mammalia
	          Amniota
	          Tetrapoda
	          Dipnotetrapodomorpha
	          Sarcopterygii
	          Euteleostomi
	          Teleostomi
	          Gnathostomata vertebrate
	          Vertebrata Metazoa
	          Craniata chordata
	          Chordata
	          Deuterostomia
	          Bilateria
	          Eumetazoa
	          Metazoa
	          Opisthokonta
	          Eukaryota
	          cellular organisms
	          root
	
	```	

	Those results give you an ideia of how your taxon of interest is hierarchically organized inside the repeat database. 
	

- You can also see the repeat library available for your species.

	`queryRepeatDatabase.pl -species cat`
	
	This will actually print the entire repeat library in fasta format, which is not very practical. To count how many entries exist, you can pipe that output and use grep to count the number of sequences.
	
	`queryRepeatDatabase.pl -species cat | grep -c ">" `

	**Output:**
	
	```
	queryRepeatDatabase
	===================
	RepeatMasker Database: RepeatMaskerLib.embl
	RepeatMasker Combined Database: Dfam_3.0
	Species: cat ( felis catus )
	Warning...unknown stuff <
	>
	782
	```
	
	The thing is: this result doesn't necessarily mean that there are 782 repetitive elements specifically for cat. Let's test it with Feliformia:

	`queryRepeatDatabase.pl -species Feliformia | grep -c ">"`
	
	**Results:**
	
	```
	queryRepeatDatabase
	===================
	RepeatMasker Database: RepeatMaskerLib.embl
	RepeatMasker Combined Database: Dfam_3.0
	Species: Feliformia ( feliformia )
	Warning...unknown stuff <
	>
	782
	```
	
Same number, right?
Here's what this script is giving you: it will use the most closely related library within the taxonomic hierarchy for that taxon. So, there's nothing specific for cat that is not present in the Feliformia library. Dr. Vanessa Gonzalez wrote this cool script that searches for all entries in the taxonomy. The script can be found in `/data/genomics/workshops/SMSC_2019/Repbase_RepeatQuery_Taxonomy_MTNT.sh` and it will output a file with a list of all taxonomy levels and the number of repeats in each library. To run this script, copy it to your `repmasker` folder and do:

`./Repbase_RepeatQuery_Taxonomy_MTNT.sh cat`

This script will output two files: `cat_tax.txt` (taxonomy) and `cat_RepBase_RepQuery.txt` (the file we actually want). If we use the command `cat` to print the contents of the file `cat_RepBase_RepQuery.txt`, here's what we will see:

**Output:**

```
Taxonomic query = 'Felis catus'
Repeats in RepBase = 782
Taxonomic query = 'Felis'
Repeats in RepBase = 782
Taxonomic query = 'Felinae'
Repeats in RepBase = 782
Taxonomic query = 'Felidae'
Repeats in RepBase = 782
Taxonomic query = 'Feliformia'
Repeats in RepBase = 782
Taxonomic query = 'Carnivora'
Repeats in RepBase = 782
Taxonomic query = 'Laurasiatheria'
Repeats in RepBase = 782
Taxonomic query = 'Boreoeutheria'
Repeats in RepBase = 1883
Taxonomic query = 'Eutheria'
Repeats in RepBase = 1888
Taxonomic query = 'Theria Mammalia'
Repeats in RepBase = 1888
Taxonomic query = 'Mammalia'
Repeats in RepBase = 1888
Taxonomic query = 'Amniota'
Repeats in RepBase = 2023
Taxonomic query = 'Tetrapoda'
Repeats in RepBase = 2023
Taxonomic query = 'Dipnotetrapodomorpha'
Repeats in RepBase = 2023
Taxonomic query = 'Sarcopterygii'
Repeats in RepBase = 2023
Taxonomic query = 'Euteleostomi'
Repeats in RepBase = 3915
Taxonomic query = 'Teleostomi'
Repeats in RepBase = 3915
Taxonomic query = 'Gnathostomata vertebrate'
Repeats in RepBase = 3915
Taxonomic query = 'Vertebrata Metazoa'
Repeats in RepBase = 3915
Taxonomic query = 'Craniata chordata'
Repeats in RepBase = 3915
Taxonomic query = 'Chordata'
Repeats in RepBase = 3915
Taxonomic query = 'Deuterostomia'
Repeats in RepBase = 3915
Taxonomic query = 'Bilateria'
Repeats in RepBase = 6244
Taxonomic query = 'Eumetazoa'
Repeats in RepBase = 6244
Taxonomic query = 'Metazoa'
Repeats in RepBase = 6244
Taxonomic query = 'Opisthokonta'
Repeats in RepBase = 6244
Taxonomic query = 'Eukaryota'
Repeats in RepBase = 6244
Taxonomic query = 'cellular organisms'
Repeats in RepBase = 6244
Taxonomic query = 'root'
Repeats in RepBase = 6244
```

As you can see, the number of repetitive elements in the library is the same until Laurasiatheria (which includes carnivorans, ungulates, shrews, bats, whales, pangolins...). Mammals are very well represented compared to other groups, so this is a good thing to keep in mind when choosing a species for the 



### 6. Run BLAT

BLAT (BLAST-like Alignment Tool, Kent 2002) is a tool that aligns DNA (as well as 6-frame translated DNA or proteins) to DNA, RNA and proteins across different species. The output of this program will also be used as hints for AUGUSTUS.

Since we don't have a transcriptome for this species, we will use one from *Taeniopygia guttata*, commonly known as zebra finch. Link [here](ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/003/957/565/GCF_003957565.1_bTaeGut1_v1.p/GCF_003957565.1_bTaeGut1_v1.p_rna.fna.gz)

<details><summary>HINT</summary>
<p>

**HINT**: Use `wget` to download the file to the `blat` folder.
`wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/003/957/565/GCF_003957565.1_bTaeGut1_v1.p/GCF_003957565.1_bTaeGut1_v1.p_rna.fna.gz`

</p>
</details>


Also, extract the file using gunzip: 

`gunzip GCF_003957565.1_bTaeGut1_v1.p_rna.fna.gz`

#### Job file: blat.job
- Queue: medium
- PE: serial
- Memory: 2G
- Module: `module load bioinformatics/blat`
- Commands:

```
blat -t=dna -q=rna ../assembly/siskin_10largest.fasta \
GCF_003957565.1_bTaeGut1_v1.p_rna.fna \
../blat/siskin_blat.psl
```

##### Explanation:
```
-t: target database type (DNA, 6-frame translated DNA or protein)
-q: query database type (DNA, RNA, protein, 6-frame translated DNA or RNA).

```

### 7. Create Augustus hints

Now we will integrate the information from the previous jobs into our AUGUSTUS job. To do that, we first need to create hint files from both RepeatMasker and BLAT. We will run the commands from this step using the **interative** queue (`qrsh`)

Also, don't forget to `cd` to your AUGUSTUS folder after logging to the interactive queue. 

For the sake of keeping things organized, let's create three folders inside your`augustus` folder: `hints`, `output` and `scaffolds`. 

<details><summary>HINT</summary>
<p>
`mkdir hints output scaffolds`

</p>
</details>
 
#### Creating hint files from RepeatMasker
In this step, we will use the .out file from the RepeatMasker use it to create a hint file for AUGUSTUS. 

- **Use the script `rmOutToGFF3.pl` to convert your .out file into GFF3**

	```
	module load bioinformatics/repeatmasker
	rmOutToGFF3.pl ../repeatmasker/siskin_10largest.fasta.out > hints/siskin_RM.gff3
	```

- **Use the script `gff2hints.pl` convert the gff3 into a hints file**
This script can be found here `/data/genomics/workshops/SMSC_2019/gff2hints.pl`. Copy it to your `augustus/hints	`
	
	Now run the script on the gff3 file you just created:
	
	`perl hints/gff2hints.pl --in=hints/siskin_RM.gff3 --source=RM --out=hints/siskin_RM_hints.out`


#### Creating hint files from BLAT
In this step, we will use the .psl file from the BLAT run to create a hint file for AUGUSTUS.

- **Sort the .psl file**  

	`cat ../blat/siskin_blat.psl | sort -n -k 16,16 | sort -s -k 14,14 > hints/siskin_blat_srt.psl`
	

- **Use the script `blat2hints.pl` from the Augustus 3.3 module**

	```
	module load bioinformatics/augustus/3.3
	blat2hints.pl --in=hints/siskin_blat_srt.psl --out=hints/siskin_blat_hints.out
	```

#### Merging hints files

To merge the hints files from RepeatMasker and BLAT, use the following command: 

`cat siskin_RM_hints.out siskin_blat_hints.out | sort -k1,1V -k4,4n > siskin_hints_RM_E.gff3`


### 8. Augustus extrinsic file

The AUGUSTUS extrinsic file has the information about the sources of evidence that will be used. In our case, we have evidence about repetitive elements (from RepeatMasker) and transcriptome (from BLAT). 

We have created a custom extrinsic file with those two sources of evidence. Copy it from `/data/genomics/workshops/genome_annot/extrinsic.M.RM.E.cfg` to YOUR augustus/config/extrinsic folder. Like this:

`cp /data/genomics/workshops/SMSC_2019/extrinsic.M.RM.E.cfg config/extrinsic`


<details><summary>EXAMPLE</summary>
<p>

##### Example of extrinsic file with RepeatMasker (RM) and BLAT (E) evidence.

```
# extrinsic information configuration file for AUGUSTUS
# 
# include with --extrinsicCfgFile=filename
# date: 15.4.2015
# Mario Stanke (mario.stanke@uni-greifswald.de)


# source of extrinsic information:
# M manual anchor (required)
# P protein database hit
# E EST/cDNA database hit
# C combined est/protein database hit
# D Dialign
# R retroposed genes
# T transMapped refSeqs
# W wiggle track coverage info from RNA-Seq

[SOURCES]
M RM E

#
# individual_liability: Only unsatisfiable hints are disregarded. By default this flag is not set
# and the whole hint group is disregarded when one hint in it is unsatisfiable.
# 1group1gene: Try to predict a single gene that covers all hints of a given group. This is relevant for
# hint groups with gaps, e.g. when two ESTs, say 5' and 3', from the same clone align nearby.
#
[SOURCE-PARAMETERS]


#   feature        bonus         malus   gradelevelcolumns
#		r+/r-
#
# the gradelevel colums have the following format for each source
# sourcecharacter numscoreclasses boundary    ...  boundary    gradequot  ...  gradequot
# 

[GENERAL]
      start      1          1  M    1  1e+100  RM  1     1    E 1    1
       stop      1          1  M    1  1e+100  RM  1     1    E 1    1
        tss      1          1  M    1  1e+100  RM  1     1    E 1    1
        tts      1          1  M    1  1e+100  RM  1     1    E 1    1
        ass      1      1 0.1  M    1  1e+100  RM  1     1    E 1    1
        dss      1      1 0.1  M    1  1e+100  RM  1     1    E 1    1
   exonpart      1  .992 .985  M    1  1e+100  RM  1     1    E 1  1e2
       exon      1          1  M    1  1e+100  RM  1     1    E 1  1e4
 intronpart      1          1  M    1  1e+100  RM  1     1    E 1    1
     intron      1        .34  M    1  1e+100  RM  1     1    E 1  1e6
    CDSpart      1     1 .985  M    1  1e+100  RM  1     1    E 1    1
        CDS      1          1  M    1  1e+100  RM  1     1    E 1    1
    UTRpart      1     1 .985  M    1  1e+100  RM  1     1    E 1    1
        UTR      1          1  M    1  1e+100  RM  1     1    E 1    1
     irpart      1          1  M    1  1e+100  RM  1     1    E 1    1
nonexonpart      1          1  M    1  1e+100  RM  1     1.15 E 1    1
  genicpart      1          1  M    1  1e+100  RM  1     1    E 1    1

#
# Explanation: see original extrinsic.cfg file
#
```

</p>
</details>

### 9. AUGUSTUS


For the AUGUSTUS job, we need the following input files:

1. assembly (masked)
2. merged RM and E hints file
3. extrinsic file
4. retraining parameters (from BUSCO)


AUGUSTUS will run serially, one scaffold at a time. In order to speed up the process, we can break the assembly into scaffolds and process them in paralel. To do so, we will use a script from EVM (EVidenceModeller) to split the assembly and the hints file, and create job arrays for AUGUSTUS.

**Partition the assembly into scaffolds** 

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


### 9a. A brief introduction to LOOPS

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


#### Combining the results and extracting fasta sequences from the gene models.

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


## Running Blast and Blast2GO
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


### 10. Split fasta into sub-files with 100 sequences
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


### 11. Create blast job array

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
This file contains all the information to run the job that is normally found in the header of a job file. You can copy a template from `data/genomics/workshops/SMSC_2019/blastp_template.conf`. Let's explore the file:

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
This file contains the job commands, as well as the modules to be loaded. You can copy the template `blastp_template.sh` from  `data/genomics/workshops/SMSC_2019/`

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

### 12. Merge the xml files

After all the jobs are done, you need to merge all xml files into one single file. You can use `cat` for that:
From the `xml` folder, do:

	cat $(find . -name "*.xml" | sort -V) > ../results/siskin_blastp_all.xml
	

### 13. Running Blast2GO

Normally, we would run Blast2go from the command line, but we won't be able to do that. Instead, we will use the GUI version to do functional annotation.

Download Blast2go from `https://www.blast2go.com/` 

We need to copy the following files from hydra to your local computer:
- siskin_augustus_all.aa
- siskin_augutus_all.gff
- siskin_blastp_all.xml



### 14. Create a genome browser with JBrowse

Now that we have an annotated genome, we can visualize the assembly and annotations using a genome browser. Today we will show you how to setup a genome browser using JBrowse, and those same files can be used with WebApollo for manual annotation.

We will use the jbrowse module on the interactive queue. 


	qrsh
	cd /pool/genomics/username/genome_annot/jbrowse
	module load bioinformatics/jbrowse/1.0


a. **prepare-refseqs.pl**: formats the reference sequence to be used with JBrowse
	
`prepare-refseqs.pl --fasta ../assembly/siskin_10largest.fasta`
	
Obs: fasta can be gzipped.  
	
b. **flatfile-to-json.pl**: format data into JBrowse JSON format from an annotation file

`flatfile-to-json.pl --gff ../augustus/output/siskin_augustus_final.gff3 --tracklabel Augustus_CDS `

Observations:  
`--gff` can't be gzipped, and must be GFF3. In addition, this script will accept `--bed` and `--gbk` (genbank) files as input.  
`--type` is the 3rd column of the GFF file. Option are: cDNA_match, CDS, exon, gene, guide_RNA, lnc_RNA, mRNA, pseudogene, rRNA, snoRNA, snRNA, transcript, tRNA, V_gene_segment.  
`--tracklabel` should be informative

c. **generate-names.pl**: builds a global index of feature names.
	

`generate-names.pl`

d. **Zip your results**

`tar -zcf siskin.tar.gz ./data`	

To visualize the results, you need to install JBrowse locally on your laptop. To make things easier, save the JBrowse folder on your Desktop. The simplified steps are listed below; you might need to install extra dependencies. 

- **From GitHub**
	
```
git clone https://github.com/GMOD/jbrowse
cd jbrowse
./setup.sh
```

- **From the JBrowse blog**: visit [http://jbrowse.org/blog](http://jbrowse.org/blog) and download one of the available zip files. Extract it and rename the folder to `jbrowse` to simplify things. The run the following command:

```
cd jbrowse
./setup.sh
```
	
After that, you need to start jbrowse by running the command `npm run start`. This will start a local jbrowse instance, and the address to access it is listed on your terminal:

```
@gmod/jbrowse@1.16.4 start /Users/mtsuchiya/Desktop/jbrowse
> utils/jb_run.js -p 8082

JBrowse is running on port 8082
Point your browser to http://localhost:8082
```

In my case, I opened a web browser (Chrome) and pasted `http://localhost:8082` on the address bar. This should start JBrowse with one of the sample projects available with the installation.

- **To visualize your data**
	- Copy the zipped file from Hydra to your machine using `scp` or Filezilla.  

		`scp username@hydra-login01.si.edu:/pool/genomics/username/smsc_2019/genome_annot/jbrowse/siskin.tar.gz ./Desktop/jbrowse`
	- Extract the file  
		`tar -zxf siskin.tar.gz`
	- Add the folder name to the address bar. In my case, this is the address to display my JBrowse files: 
		`http://localhost:8082/index.html?data=siskin`
		
		Let me explain what this address mean:  
			- `localhost:8082`: this is the port in your computer that's being used by JBrowse.   
			- `index.html`: you will find this file in your local `jbrowse` folder. This is a HTML page, formatted to display the JBrowse files correctly.  
			- `data=siskin`: this is the path to your data file inside the `jbrowse` folder. The examples provided with the installation are inside `sample_data/json/volvox`. If you replace your folder name by this, you will be able to see the sample data for *Volvox*. 

