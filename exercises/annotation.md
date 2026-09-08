# CL5: Whole-Genome Assembly

In this tutorial, we are going to assemble an unknown genome.

---
## 🧠 Learning Objectives

By the end of this exercise, you should be able to:
- Assemble a genome
- Assess the quality of a genome assembly

## ⛑️ Setting Up Working Environment

Log in on your LEAP2 account. Create a new `conda` environment for genome assembly: 

```bash
conda create -n assembly -c bioconda -c conda-forge spades seqkit
```

[Anvi'o](https://anvio.org/) is a comprehensive open-source analysis and visualization platform for microbial omics. We will be using it throughout the course. Create a `conda` environment for Anvi'o:

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
conda deactivate
```

## 🧑🏻‍💻 Working Directory and Data

Go to your `microbial_genomics` directory and, in there, create a working directory for genome assembly:

```bash
cd </path/to/>microbial_genomics
mkdir assembly
```

Create a working directory for your sequence data in there:

```bash
cd assembly
mkdir data
```

Download the two fastq files, `Unknown_R1.trimmed.fastq.gz` and `Unknown_R2.trimmed.fastq.gz` from Canvas to the local computer, and upload them into your data directory above on LEAP2.

Genomic DNA from a bacterial isolate was sequenced on an Illumina NextSeq platform, with read pairs of 150 bp eacg, with an insert size of 350 bp. As you can tell from the file names, the data have already been QCed and trimmed.

>[!NOTE]
> We are skipping QC and trimming for the sake of time.

## 🧩 Assembly

[SPAdes](https://ablab.github.io/spades/) is a very versatile genome assembler and easy to use.  We will use it for this session:

```bash
cd </path/to/assembly>
mkdir spades
cd spades
```

A symbolic link (aka symlink or soft link) is a special type of file that points to another file or directory. They are a common way to create shortcuts to easily access files at other locations in a same file system. Create a symlinks to the data:

```bash
ln -s ../data/Unknown_R*.fastq.gz .
```
Activate the Conda environment:

```bash
conda activate assembly
```

The documentation of SPAdes can be found [here](https://ablab.github.io/spades) or by running `spades.py -h`. 

These are the steps SPAdes follows: 

<img width="265" height="355" alt="Screenshot 2025-09-18 at 10 42 34 AM" src="https://github.com/user-attachments/assets/9ad8209b-6707-499e-b627-594461cb6f39" />
<img width="479" height="267" alt="Screenshot 2025-09-18 at 10 43 29 AM" src="https://github.com/user-attachments/assets/f65ab785-ac47-4846-88a5-3ab6be2075f4" />

The SPAdes manual makes a few recommendations:

1. Because we are working with the genome of a bacterial isolate genome (and not a metagenome), the read coverage ais likely pretty high (>50x). We need to run our assembly in the `--isolate` mode.
2. For 150 bp reads, they recommend *k*-mers of 21, 33, 55, and 77 bp.

> Small *k* → better sensitivity, connects through low-coverage regions but more likely to introduce tangles/repeats.
> Large *k* → more specificity, helps resolve repeats, but risks breaking contigs in low-coverage areas.
> Rule of thumb 👍🏼: the largest *k* should be about ½ the read length or a bit more.

You may ask, why the odd numbers? 
--> With an even *k*, you can end up with *k*-mers that are perfect palindromes, i.e., a sequence that reads the same in the forward strand and in the reverse complement of the sequence.

```
                5' - TCGCGA - 3'
                3' - AGCGCT - 5'
                5' - TCGCGA - 3'

                5' - TCGCG - 3'
                3' - AGCGC - 5'
                5' - CGCGA - 3'
```
        
Create a script for SPAdes:

```bash
vim spades.sh
```

Copy-paste the following script:

```bash
#!/bin/bash
#SBATCH --job-name=spades
#SBATCH --partition=shared
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --time=1:00:00
#SBATCH --mem=20G

# Get started
echo "Job started on $(hostname) at $(date)"

source ~/.bashrc
conda activate assembly

#Variables
export R1=Unknown_R1.trimmed.fastq.gz
export R2=Unknown_R2.trimmed.fastq.gz
export OUTDIR=output

#Commands
spades.py --isolate -1 $R1 -2 $R2 -o $OUTDIR -t $SLURM_NTASKS -k 21,33,55,77

# Finish up
conda deactivate

echo "Job Ended at $(date)"
```

Save the script and run it:

```bash
sbatch spades.sh
```

This should take <5 min. Keep track of the job using `squeue`. Once it finishes, inspect the SLURM output file and try to make sense of the steps taken:

```bash
less slurm-<jobID>.out
```

Deactivate the Conda environment and go back to the `assembly` directory:

```bash
conda deactivate
cd ..
```

## ✅ Assembly QC Assessment

We will use [Quast](https://quast.sourceforge.net/index.html) to asses quality of the resulting genome assembly. Because of incompatibilities with the assembly environment, we will create a separate environment for quast:

```bashz
conda create -n quast python=3.7
conda activate quast
conda install -c bioconda quast
conda deactivate
```

Create a symlink to the contigs assembled by SPAdes:

```bash
ln -s ../spades/output/contigs.fasta
```

Run Quast on an interactive session:

```bash
sinteractive -p shared -n 1 --mem-per-cpu=10G --time=1:00:00
quast.py contigs.fasta
exit
```

Once finished, download the HTML report (`quast_results\results_<...>\report.html`) to the local computer. It should be similar to [this one](https://htmlpreview.github.io/?https://github.com/Goch-Lab/TXST_Microbial-Genomics_2026/blob/main/data/04_assembly/report.html).

Now we’re going to put our genome assembly into the anvi’o framework and begin to look at our assembled genome.

Deactivate your conda environment.
```
conda deactivate
```

## 🧪 Exercise 3: Anvi'o

First, let's move back to our working_dir and make a new directory for anvi'o.
```bash
cd ../
pwd
mkdir anvio
cd anvio
```

Symlink the contigs.fasta and fastq files here:
```
ln -s ../spades/output/contigs.fasta .
ln -s ../../data_dir/*.fastq.gz .
```

Activate the conda environment.
```
conda activate anvio-8
```

For us to get our assembly into anvi’o, first we need to generate what it calls a contigs database using the `anvi-gen-contigs-database ` command. This will organize our contigs in an anvi’o-friendly way, and provide information about them. 

When run on `contigs.fasta` this program will:

1. Compute k-mer frequencies for each contig (the default is 4, but you can change it using --kmer-size parameter if you feel adventurous).
2. Soft-split contigs longer than 20,000 bp into smaller ones (you can change the split size using the --split-length flag). When the gene calling step is not skipped, the process of splitting contigs will consider where genes are and avoid cutting genes in the middle. For very, very large assemblies this process can take a while, and you can skip it with --skip-mindful-splitting flag.
3. Identify open reading frames using Prodigal, UNLESS, (1) you have used the flag --skip-gene-calling (no gene calls will be made) or (2) you have provided external-gene-calls.

You can see what the command needs and options you want to set by `anvi-gen-contigs-database -h `.

```bash
anvi-gen-contigs-database -f contigs.fasta -o contigs.db -n unknown_genome
```

```bash
anvi-run-hmms -c contigs.db -I Bacteria_71 -T 4
```
```bash
anvi-run-hmms -c contigs.db -I Ribosomal_RNA_16S -T 4
```

Anvio will determine our genome completeness and contamination with single-copy genes (SCGs) by running this:
```
anvi-estimate-genome-completeness -c contigs.db
```

So you can see, our genome is 100% complete based on the presence of expected SCGs. The redundancy is ~4%, so just below the 5% threshold for high-quality genomes. 

Now, let's assign functions to our ORFS!
First, we will do so with the NCBI COG database. We have to run `anvi-setup-ncbi-cogs` to download and set up the database. You only have to do this once. 

```bash
anvi-setup-ncbi-cogs -T 4 
anvi-run-ncbi-cogs -c contigs.db -T 4
```

Next, we will also assign function using the KEGG database. 
```bash
anvi-run-kegg-kofams -c contigs.db -T 4
```
This will take ~8-10 min.

When this next program runs, it will look at the KOfam annotations (for KEGG Orthologs, or KOs) within each genome, match them up to the KEGG module definitions to estimate the completeness of each module/pathway. 
A module is considered ‘complete’ or ‘present’ in a genome if its completeness score is above a certain threshold, which can be set with the --module-completion-threshold parameter. A static threshold such as this is not the most ideal metric, especially since metabolic modules have variable numbers of genes - for example, with the default threshold of 0.75 (75%), a module with 3 KOs in it would only be considered complete if all 3 of those KOs were found in a genome, while a module with 5 KOs could be considered complete if only 4 of its KOs were found. But it is what it is without diving deeper and doing additional checks (: 
```
anvi-estimate-metabolism -c contigs.db
```

We can then integrate mapping information from recruiting our reads to the assembly. This mapping information is placed into anvi’o with the anvi-profile program, which generates another type of database anvi’o calls a “profile database”. In contrast to the contigs-db, an anvi’o single-profile-db stores sample-specific information about contigs. Profiling a BAM file with anvi’o using anvi-profile creates a single profile that reports properties for each contig in a single sample based on mapping results. 

SAM files are a type of text file format that contains the alignment information of various sequences that are mapped against reference sequences. BAM files contain the same information as SAM files, except they are in binary file format which is not readable by humans. On the other hand, BAM files are smaller and more efficient for software to work with than SAM files, saving time and reducing costs of computation and storage. 

Run these line by line. 
```bash
bowtie2-build contigs.fasta contigs.btindex

bowtie2 -q -x contigs.btindex \
        -1 unknown_R1_paired.fastq.gz \
        -2 unknown_R2_paired.fastq.gz \
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








