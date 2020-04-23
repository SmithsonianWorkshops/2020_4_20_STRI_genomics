
# Genome annotation - STRI 2020 (Day 1)

<!-- TOC depthFrom:2 -->

 * [Folder structure](#Folder-structure)
 * [Input file](#Input-file)
 * [Run BUSCO using the --long flag](#Run-BUSCO-using-the---long-flag)
 	* [Using BUSCO output for the AUGUSTUS run](#Using-BUSCO-output-for-the-AUGUSTUS-run)
 * [Masking and annotating repetitive elements](#3-Masking-and-annotating-repetitive-elements)
 * [Run BLAT](#4-Run-BLAT)
 * [Create Augustus hints](#Create-Augustus-hints)
 	* [Creating hint files from RepeatMasker](#Creating-hint-files-from-RepeatMasker)
	* [Creating hint files from BLAT](#Creating-hint-files-from-BLAT)
	* [Merging hints files](#Merging-hints-files)
 * [Edit the extrinsic file to add only the evidence you have](Augustus-extrinsic-file)


<!-- /TOC -->

### Folder structure

Now let's create our folder structure for this workshop. It's easier to troubleshoot any issues if we are all working within the same framework. First, we will create a folder called `genome_annot` in your `/scratch/genomics/username` folder. 

```
cd /scratch/genomics/your_username/stri2020

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


### Input file
For this session, we will use the 10 largest scaffolds from the siskin assembly. You can find this file here: `/data/genomics/workshops/STRI2020/siskin_10largest.fasta`

Copy this file to your assembly folder.

---
**Extracting the largest sequences**
<details><summary>SOLUTION</summary>
<p>

To generate this file, we used `bioawk` and `samtools` to extract the sequences from the original assembly:

`module load bioinformatics/bioawk`
`module load bioinformatics/samtools`

Create a list with the 10 largest sequences. The number of sequences is determined by the number following the `head` command in the end. 

`cat assembly.fasta | bioawk -c fastx '{ print length($seq), $name }' | sort -k1,1rn | head -10 > 10largest.list`

Use `samtools` to extract the list of sequences from the original assembly:

`xargs samtools faidx assembly.fasta < 10largest.list > assembly_10largest.fasta` 

</p>
</details>

---



### Run BUSCO using the --long flag

BUSCO (Simão et al. 2015; Waterhouse et al. 2017) assesses completeness by searching the genome for a selected set of single copy orthologous genes. There are several databases that can be used with BUSCO and they can be downloaded from here: [https://buscos.ezlab.org](https://buscos.ezlab.org). 

---
##### *** Before running BUSCO, copy the file augustus/config folder to a place where you have writing privileges***
The AUGUSTUS config folder can be found here:  `/share/apps/bioinformatics/augustus/conda/3.3.2/config/`. Copy it to the folder `augustus` inside your `genome_annot` folder.

<details><summary>SOLUTION</summary>
<p>
Assuming you are in the folder `/scratch/genomics/username/stri2020/genome_annot/augustus`:  

`cp -r /share/apps/bioinformatics/augustus/conda/3.3.2/config/ .`

</p>
</details>

---

**Database:**

For this workshop, we will use the Aves database. Download it to the folder `busco` using the command `wget` and extract it. Let's `cd` to the directory `busco` first.

	wget https://buscos.ezlab.org/datasets/aves_odb9.tar.gz
	tar -zxf aves_odb9.tar.gz


According to the BUSCO manual, the `--long` flag turns on Augustus optimization mode for self-training. It can be used as a training set for AUGUSTUS.


#### Job file 1: busco_siskin.job

- PE: multi-thread
- Number of CPUs: 4
- Memory: 6G (6G per CPU, 24G total)
- Module: `module load bioinformatics/busco`
- Commands:

```
export AUGUSTUS_CONFIG_PATH="/scratch/genomics/username/stri2020/genome_annot/augustus/config"
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


##### Using BUSCO output for the AUGUSTUS run
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


### Masking and annotating repetitive elements

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
Here's what this script is giving you: it will use the most closely related library within the taxonomic hierarchy for that taxon. So, there's nothing specific for cat that is not present in the Feliformia library. Dr. Vanessa Gonzalez wrote this cool script that searches for all entries in the taxonomy. The script can be found in `/data/genomics/workshops/STRI2020/Repbase_RepeatQuery_Taxonomy_MTNT.sh` and it will output a file with a list of all taxonomy levels and the number of repeats in each library. To run this script, copy it to your `repmasker` folder and do:

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



### Run BLAT

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

### Create Augustus hints

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
This script can be found here `/data/genomics/workshops/STRI2020/gff2hints.pl`. Copy it to your `augustus/hints	`
	
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


### Augustus extrinsic file

The AUGUSTUS extrinsic file has the information about the sources of evidence that will be used. In our case, we have evidence about repetitive elements (from RepeatMasker) and transcriptome (from BLAT). 

We have created a custom extrinsic file with those two sources of evidence. Copy it from `/data/genomics/workshops/genome_annot/extrinsic.M.RM.E.cfg` to YOUR augustus/config/extrinsic folder. Like this:

`cp /data/genomics/workshops/STRI2020/extrinsic.M.RM.E.cfg config/extrinsic`


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

