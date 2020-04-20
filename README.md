# 2020_4_20_STRI_genomics

## Introduction to Genome Assembly and Annotation

The improvement in DNA sequencing technologies over the last three decade have led to new opportunities for studying organisms at the genomic level. In fact, the past several years have seen remarkable progress in understanding genomic variation within species and to identify the genes underliying phenotipic variation, many genes with clear links to adaptive differences in nature. By the end of this workshop I hope you will have an idea of:

1. What are the aims of a Genome Assembly and Annotation project?
2. Which are good practices in a Genome Assembly and Annotation?
3. Why Genome Assembly and Annotatino are difficult and which are their challanges?

### What are the aims of a Genome Assembly and Annotation project?

1. Define the complete genome sequence of an organism
2. Define the structure and organization of the genome (# of chromosomes, Annotate genes and important funtional features)

### Which are good practices in a Genome assembly and Annotation?

1. Investigate the properties of the genome you study
  * Genome size
  * Repeats
  * Heterozygosity
  * Ploidy level
  * GC
  
2. Start with high quality DNA 
  * High Molecular Weight DNA (>50Kb)
  * Chemical Purity (Reduce as possible any carry-over contamintats)

3. Choose an appropriate sequencing technology (Understand the pro and cons of the the different sequencing tecnologies), think about: 

  * Read size (Long reads [Pacbio, ONT] vs Short reads [Illumina])
  * Error rate
  
4. Estimate computational resources need it for Genome Assembly and Annotation (explore possible software and their pro and con)

### Why Genome Assembly is difficult and which are their challanges?

1. Polimorphism (heterozigocity)
 * Ploidy of the organims (diploid, tetraploid etc)
 * Chimera: An organism with multiple genomes in the same sample
 * Variation across homologous regions on chromosomes may be different
 * Pools of individuals: some organism are two small to obtain enogh DNA for library prep.
 
2. Repeats
 * Short Interspersed Elements
 * Long Intersepesed Elements
 * Duplications
 * Poliploidy
 
3. Sample Contamination
 * Samples can get contaminates from different sources (Human, Bactera or other Endo or ecto symbionts).
 
4. Errors related with the sequencing technology.
 * Phred Scores (Logarithmically linked to probablility of erro)
   - Q10: p = 1 in 10 of wrong base call
   - Q20: p = 1 in 100 of wrong base call
   - Q30: p = 1 in 1000 of wrong base call
   
   
