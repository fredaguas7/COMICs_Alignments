# COMICs_Allignments

In this repo you will find the tools we usually run for alignments and pseudo-alignments.

# Alignments - STAR

For alignments to the genome we generally use STAR.
You should also trim the reads for QC, which can be done with trim_galore.

## Installation 

You can set up the conda environments like this:
````
conda create -n trim_galore
conda activate trim_galore
conda install bioconda::trim-galore

conda create -n STAR
conda activate STAR
conda install bioconda::star
`````

## Trim the reads 

Trim_galore trims the read adapters and also poor quality reads and is simple to run:

````
trim_galore  --paired R1.fq.gz R2.fq.gz 
````

## Create the genome index 

Get the fasta and annotation (gtf preferentially, will also work with gff3 but you may have troubles ahead) files for your genome of interest and generate the index:

````
STAR --runThreadN 8  \
--runMode genomeGenerate \
--genomeDir  Genomes/My_Genome_STAR \
--genomeFastaFiles Genomes/My_Genome.fa \
--sjdbGTFfile Genomes/My_Genome.gtf  \
--sjdbOverhang 100 
````

## Allign the reads

We use the options --outFilterMultimapNmax 1 for uniquely mapped reads and --quantMode GeneCounts --sjdbGTFfile GTF to get the gene counts.

````
STAR --runThreadN 8 \
    --genomeDir  GENOME \
    --outFileNamePrefix PREFIX \
    --readFilesCommand gunzip -c \
    --outSAMtype BAM SortedByCoordinate \
    --outFilterMultimapNmax 1 \
    --readFilesIn R1_trimmed.fq.gz R2_trimmed.fq.gz \
    --quantMode GeneCounts --sjdbGTFfile GTF
````

## In case of small genomes/genomes with a lot of scaffolds

When creating the index you should consider this options 

````
# Scale down --genomeSAindexNbases to min(14, log2(GenomeLength)/2- 1)
# Scale down --genomeChrBinNbits to min(18,log2[max(GenomeLength/NumberOfReferences,ReadLength)])

STAR --runThreadN 8  \
--runMode genomeGenerate \
--genomeDir  Genomes/My_Genome_STAR \
--genomeFastaFiles Genomes/My_Genome.fa \
--sjdbGTFfile Genomes/My_Genome.gtf  \
--sjdbOverhang 100 --genomeSAindexNbases 14 \
--genomeChrBinNbits 12

````

# Pseudo-alignment - kallisto

For pseudo-alignment we usually run kallisto. We will also use trim_galore as previously.

## Installation

````
conda create -n kallisto
conda activate kallisto
conda install bioconda::kallisto
````

## Trim the reads 

First step as in the regular alginments:

````
trim_galore  --paired R1.fq.gz R2.fq.gz 
````

## Create the transcriptome index 

For kallisto you will need the transcriptome fasta:

````
kallisto index -i Genomes/my.transcriptome.idx Genomes/my.transcripts.fa
````

## Allign to transcriptome

There are some options you should keep in mind for kallisto

````
# Mind if the data is unstranded or stranded, and if the R1 is forward or reverse
--unstranded
--rf-stranded
--fr-stranded

# If the data is single end you have to pass the fragment lenght and standard deviation. On pair data these are infered from the data
--single
-l 250
-s 50

# Use more threads to speed up (even more!)
-t 8

````
Your command can look something like this:

````
kallisto quant -i Genomes/my.transcriptome.idx -o output/dir/ --rf-stranded -t 8 R1_trimmed.fq.gz R2_trimmed.fq.gz
````
