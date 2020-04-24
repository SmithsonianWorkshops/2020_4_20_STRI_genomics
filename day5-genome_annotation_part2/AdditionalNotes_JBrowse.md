### Create a genome browser with JBrowse

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
	- Copy the zipped file from Hydra to your machine using `scp`, Filezilla or Firefox Send (if you're using telework)  

		`scp username@hydra-login01.si.edu:/pool/genomics/username/smsc_2019/genome_annot/jbrowse/siskin.tar.gz ./Desktop/jbrowse`
	- Extract the file  
		`tar -zxf siskin.tar.gz`
	- Add the folder name to the address bar. In my case, this is the address to display my JBrowse files: 
		`http://localhost:8082/index.html?data=siskin`
		
		Let me explain what this address mean:  
			- `localhost:8082`: this is the port in your computer that's being used by JBrowse.   
			- `index.html`: you will find this file in your local `jbrowse` folder. This is a HTML page, formatted to display the JBrowse files correctly.  
			- `data=siskin`: this is the path to your data file inside the `jbrowse` folder. The examples provided with the installation are inside `sample_data/json/volvox`. If you replace your folder name by this, you will be able to see the sample data for *Volvox*. 




  
