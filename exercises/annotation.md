# CL6: Genome Annotation

In this tutorial, we are going to annotate the genome assembly generated last session.

---
## 🧠 Learning Objectives

By the end of this exercise, you should be able to:
- Predict genes in a genome
- Annotate functions of different type of genetic elements

## ⛑️ Setting Up Working Directory and Environment

Go to your `microbial_genomics` directory and, in there, create a working directory for genome annotation:

```bash
cd </path/to/>microbial_genomics
mkdir annotation
cd annotation
```

[Anvi'o](https://anvio.org/) is a comprehensive open-source analysis and visualization platform for microbial omics. We will be using it throughout the course. Create a Conda environment for Anvi'o:

```bash
conda deactivate
conda remove -n anvio-9 --all -y
conda create -y --name anvio-9 python=3.10
conda activate anvio-9
conda install -y -c conda-forge -c bioconda python=3.10 \
        sqlite=3.46 prodigal idba mcl muscle=3.8.1551 famsa hmmer diamond \
        blast megahit bowtie2 bwa graphviz "samtools>=1.9" \
        trimal iqtree trnascan-se fasttree r-base r-tidyverse \
        r-optparse r-stringi r-magrittr bioconductor-qvalue meme ghostscript \
        nodejs=20.12.2 llvmlite numba
conda install -y -c bioconda fastani
conda install -y -c conda-forge -c bioconda spades
conda install -y -c conda-forge -c bioconda vmatch
curl -L https://github.com/merenlab/anvio/releases/download/v9/anvio-9.tar.gz --output anvio-9.tar.gz
pip install anvio-9.tar.gz
```

Create symlinks to the sequence data and the genome assembly:

```bash
ln -s ../assembly/data/*.fastq.gz .
ln -s ../assembly/spades/output/contigs.fasta .
```

## 🧪 Exercise 3: Annotation

To make our contigs "more easily accessible" for Anvi'o, we can generate a contig database using the command `anvi-gen-contigs-database`. Look  at the command usage:

```bash
anvi-gen-contigs-database -h
```

In short, this command:

1. Computes *k*-mer frequencies for each contig (the default is 4, but you can change it using the `--kmer-size` parameter).
2. Soft-splits contigs longer than 20,000 bp into smaller ones (you can change the split size using the `--split-length` flag). When the gene-calling step is not skipped, the process of splitting contigs will take into account the location of genes and avoid cutting genes in the middle. For large assemblies, this process can take a while and you can skip it with the `--skip-mindful-splitting` flag.
3. Identifies open reading frames using Prodigal, UNLESS:
   - You have used the flag `--skip-gene-calling` (no gene calls will be made), or
   - You have provided `external-gene-calls`.

Let's run the annotation with Anvi'o on an interactive shell:

```bash
sinteractive -p shared -n 4 --mem-per-cpu=5G --time=2:00:00
```

Create the contig database:

```bash
conda activate anvio-9
anvi-gen-contigs-database -f contigs.fasta -o contigs.db -n unknown_genome
```

Search for open reading frames (ORFs) using a set of 71 single-copy genes (SCGs) that are core to all bacteria:

```bash
anvi-run-hmms -c contigs.db -I Bacteria_71 -T 4
```

Search for the *16S* rDNA gene:

```bash
anvi-run-hmms -c contigs.db -I Ribosomal_RNA_16S -T 4
```

Anvi'o determines genome completeness and contamination with SCGs by running this:

```
anvi-estimate-genome-completeness -c contigs.db
```

As you can see, our genome is 100% complete based on the presence of expected SCGs. The redundancy is ~4%, so just below the 5% threshold for high-quality genomes. 

Let's assign functions to our ORFs. First, we will do so with the NCBI COG database. Download and set up the database: 

```bash
anvi-setup-ncbi-cogs -T 4 
anvi-run-ncbi-cogs -c contigs.db -T 4
```

Next, assign functions using the KEGG database:

```bash
anvi-setup-kegg-data -T 4
anvi-run-kegg-kofams -c contigs.db -T 4
```

This will take ~8-10 min. Next, let's estimate KEGG pathways completeness:

```bash
anvi-estimate-metabolism -c contigs.db
```

This program looks at the KOfam annotations (for KEGG Orthologs, or KOs) within each genome, and matches them up to the KEGG module definitions to estimate the completeness of each module/pathway. A module is considered ‘complete’ or ‘present’ in a genome if its completeness score is above a certain threshold, which can be set with the `--module-completion-threshold` parameter. A static threshold such as this is not the most ideal metric, especially since metabolic modules have variable numbers of genes. For example, with the default threshold of 0.75 (75%), a module with 3 KOs in it would only be considered complete if all 3 of those KOs were found in a genome, while a module with 5 KOs could be considered complete if only 4 of its KOs were found.

We can then integrate mapping information from aligning our reads to the assembly. This mapping information is placed into Anvi’o with the `anvi-profile` program, which generates another type of database Anvi’o calls a “profile database”. In contrast to the contigs-db, an Anvi’o single-profile-db stores sample-specific information about contigs. Profiling a BAM file using `anvi-profile` creates a single profile that reports properties for each contig in a single sample based on mapping results. 

SAM files are a type of text file format that contains the alignment information of various sequences that are mapped against reference sequences. BAM files contain the same information as SAM files, except they are in binary file format which is not readable by humans. On the other hand, BAM files are smaller and more efficient for software to work with than SAM files, saving time and reducing costs of computation and storage. Run these line by line:

```bash
bowtie2-build contigs.fasta contigs.btindex

bowtie2 -q -x contigs.btindex \
        -1 Unknown_R1_paired.fastq.gz \
        -2 Unknown_R2_paired.fastq.gz \
        -p 4 -S unknown_assembly.sam

samtools view -bS unknown_assembly.sam > unknown_assembly.bam

anvi-init-bam unknown_assembly.bam -o unknown_anvio.bam

anvi-profile -i unknown_anvio.bam -c contigs.db -T 4 --cluster-contigs -o unknown_profiled/

```

Let's export a fasta file of Prodigal-identified open-reading frames. 
```bash
anvi-get-sequences-for-gene-calls -c contigs.db -o gene_calls.fa
```

And do the same for our 16S rRNAs, SCGs, COGs, and KEGGs. Run each one line by line. 
```bash
anvi-get-sequences-for-hmm-hits -c contigs.db --hmm-source Ribosomal_RNA_16S -o rRNAs.fa
anvi-get-sequences-for-hmm-hits -c contigs.db --hmm-sources Bacteria_71 --get-aa-sequences -o bacterial_SCGs.faa --no-wrap
anvi-export-functions -c contigs.db -o cog_functions.txt --annotation-sources COG20_CATEGORY
anvi-export-functions -c contigs.db -o kegg_functions.txt --annotation-sources KEGG_Class,KOfam
```

Now we are going
```bash
anvi-script-add-default-collection -p unknown_profiled/PROFILE.db
anvi-summarize -c contigs.db -p unknown_profiled/PROFILE.db -C DEFAULT -o unknown_assembly_summary/
anvi-interactive -c contigs.db -p unknown_profiled/PROFILE.db --title "Unknown assembly"
```
Anvi'o visualization is really geared towards metagenomics/comparative genomics like so:

<img width="1062" height="1066" alt="image" src="https://github.com/user-attachments/assets/5f4be3be-55ab-4d8f-a502-7467763054b6" />


## 📝 Assignment due next class on Canvas
1. Who does your genome belong to? (Hint: what can you do with the information in rRNAs.fa file)
2. Tell me about the organism (environments it's found in, metabolisms, etc). Provide literature references. 
3. Using the provided cog_functions.txt, make a bar chart of COG functional categories. Count the number of genes per COG category letter (A, C, E, …). If a gene is annotated to multiple categories (entries separated by !!!), count it once for each category it belongs to. Label axes and add category names. Feel free to use Excel (easy) or R (advanced). If you use ChatGPT, provide a screenshot of the solution you used. 
Example:
<img width="1000" height="500" alt="image" src="https://github.com/user-attachments/assets/8f0804d9-c542-45f6-9c4b-780c4f335de7" />








