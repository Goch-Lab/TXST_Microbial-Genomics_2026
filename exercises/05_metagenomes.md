# CL11: Metagenomics I: Assembly and Binning

This tutorial is based on the [metagenomics tutorial](https://github.com/Penn-State-Microbiome-Center/KickStart-Workshop-2026/tree/main/Day3-Shotgun) of the KickStart Workshop from the [Penn State One Health Microbiome Center](https://www.huck.psu.edu/research/centers-institutes/one-health-microbiome-center). It will cover a few of the basic computational approaches to studying WGS metagenomic data. In contrast to *16S* amplicon sequencing, there is no agreed-upon "all-in-one" analysis platform for WGS metagenomic analysis. Because of that, we will be covering some of the state-of-the-art stand-alone tools [according to the Initiative for the Critical Assessment of Metagenome Interpretation (CAMI)](https://doi.org/10.1038/s41592-022-01431-4).

<img width="957" height="718" alt="image" src="https://github.com/Goch-Lab/TXST_Microbial-Genomics_2026/blob/main/data/CL11/128754520-4e2852aa-52b4-43a5-9e68-4e6f4f030379.png" />

---
## 🧠 Learning Objectives

By the end of this exercise, you should be able to:

- Distinguish between genome and metagenome assembly.
- Understand how genome binning works. 

## Metagenome Assembly

Metagenome assembly is one of the most computationally intensive part of WGS metagenomic analysis. This is due in part to how tangled the assembly graphs can look. For an illustration, here is a de Bruijn graph with a *k*-mer size of 50 for a mock metagenome consisting of 113 microorganisms:

<img width="1080" height="1085" alt="image" src="https://github.com/Goch-Lab/TXST_Microbial-Genomics_2026/blob/main/data/CL11/128256031-fb788323-583b-41b8-b02c-8c0a2ed86d74.png" />

Disentangling such a big knot into linear contigs requires complex algorithmic approaches. One of the most successful current approaches is to construct de Bruijn graphs with multiple *k*-mer sizes. However, metagenome assembly remains a field of active development. Since we don't want to wait hours to days to complete the assemblies, we will be using small datasets. This might give you the impression that the tools are not resource intensive, but do not be deceived! The amount of resources required does not scale linearly with the number of reads (it grows much faster than that).

In this tutorial, we will use the best-performing assembler from the CAMI2 competition: [MEGAHIT](https://github.com/voutcn/MEGAHIT). MEGAHIT is a *de novo* assembler first introduced in 2015. It utilizes multiple *k*-mer sizes when building a de Bruijn graph along with a strategy to rescue low-coverage regions while attempting to account for sequencing errors:

<img width="385" height="440" alt="image" src="https://github.com/Goch-Lab/TXST_Microbial-Genomics_2026/blob/main/data/CL11/bioinformatics_31_10_1674_f2.gif" />

Login onto LEAP2 and create a working directory in your `microbial genomics` directory:

```bash
cd <path/to/microbial_genomics>
mkdir metagenome_assembly
cd metagenome_assembly
```

Create a `data` directory and download and decompress the data:

```bash
mkdir data
cd data
wget -i https://raw.githubusercontent.com/Penn-State-Microbiome-Center/KickStart-Workshop-2022/main/Day5-Shotgun/Data/file_list.txt
ls *.gz | xargs -P6 -I{} gunzip {}
```

Create a Conda environment for the metagenomics tutorials and install MEGAHIT in there:

```bash
conda create -n metagenomics bioconda::megahit
conda activate metagenomics
megahit -h
conda deactivate
cd ..
```

There are a variety of parameters that can be specified with MEGAHIT. However, the main ones we will focus on specify if the input data (which must be fasta or fastq) is paired end in separate (`-1` and `-2` flags) or interleaved (`-12` flag) files, or `-r` single-end, as well as the specification of the output directory with `-o`.

Run MEGAHIT using the default parameters on one of the samples from an interactive shell:

```bash
sinteractive -p shared -n 4 --mem-per-cpu=10G --time=2:00:00
conda activate metagenomics
mkdir output
megahit -r data/SRS014464-Anterior_nares.fasta -o output/default
```

Before examining the output, let's run it again with different settings. We can take into account the graphs from all *k*-mer sizes by setting the minimum *k*-mer count to 1:

```bash
megahit -r data/SRS014464-Anterior_nares.fasta -o output/min1 --min-count 1
```

This will likely result in many more shorter contigs due to trusting every *k*-mer as informative. In other words, since we have ignored the effect of noise, we will likely have a range of contigs that only differ by a few bases, which are likely result of sequencing errors.

Alternatively, we could change the range of *k*-mer sizes to use. In general, the larger the *k*-mer size, the more specific (and less sensitive) the assembly will be. In practice, this can result in the assembly of high abundance organisms. Inversely, the smaller the *k*-mer size, the more sensitive (but less specific) the assembly will be (i.e., you may get a bunch of really short contigs):

```bash
megahit -r data/SRS014464-Anterior_nares.fasta -o output/ksize15-51-10 --k-min 15 --k-max 51 --k-step 10
```

## 🧪 Step 2: Gene presence-absence

First, this data was prepared with an older version of Anvi'o, so we need to migrate the data to work with our version like so:
```bash
anvi-migrate --migrate-safely *.db
```

Let’s take a first look at the dataset and put our _E. faecalis_ contigs in the context of 20 infant gut metagenomes.
```bash
anvi-interactive -p PROFILE.db -c CONTIGS.db
```

We have some additional data to add that tells us more about the sample:
```bash
anvi-import-misc-data additional_layers_table.txt -p PROFILE.db -t layers
```
```bash
anvi-interactive -p PROFILE.db -c CONTIGS.db
```

Instead of doing this per-contig basis, we can look at the distribution patterns of genes. Don't worry, you will see warnings because it's our first time running this, and it needs to generate the gene database for us.
```bash
anvi-interactive -p PROFILE.db -c CONTIGS.db -C DEFAULT -b EVERYTHING --gene-mode
```
If you right-click any of the genes, you can see that this menu has more options, including viewing gene context. 


Let's output some data files on gene coverage and detection for you to use for the assignment:
```bash
anvi-export-gene-coverage-and-detection -c CONTIGS.db -p PROFILE.db -O efae
```

## 🧪 Step 3: SNVs

Now let's assess SNVs. As you can imagine, there are a lot of SNVs across all this data. We can get some context by looking at particular genes. 
```bash
anvi-interactive -p PROFILE.db -c CONTIGS.db -C DEFAULT -b EVERYTHING --gene-mode
```

We can export the data, but it would be just too much. So instead let's look at SAAVS (perhaps a bit more meaningful). 
```
anvi-gen-variability-profile -c CONTIGS.db -p PROFILE.db -C DEFAULT -b EVERYTHING --engine AA --min-coverage-in-each-sample 5 --quince-mode --compute-gene-coverage-stats -o snvs.txt 
```

Some info on settings:
* --engine = will define the output profile you will get from this program. The engine can focus on nucleotides (NT), codons (CDN), or
  an amino acids (AA).
* --min-coverage-in-each-sample = Minimum coverage of a given variable nucleotide position in all samples. If a nucleotide position is covered less than this value even in one
                        sample, it will be removed from the analysis. Default is 0.
* --quince-mode = The default behavior is to report allele frequencies only at positions where variation was reported during profiling (which by default uses
                        some heuristics to minimize the impact of error-driven variation). So, if there are 10 samples, and a given position has been reported as a
                        variable site during profiling in only one of those samples, there will be no information will be stored in the database for the remaining 9.
                        When this flag is used, we go back to each sample, and report allele frequencies for each sample at this position, even if they do not vary.
                        It will take considerably longer to report when this flag is on, and the use of it will increase the file size dramatically, however it is
                        inevitable for some statistical approaches and visualizations.
* --compute-gene-coverage-stats = If provided, gene coverage statistics will be appended for each entry in variability report. This is very useful information, but will not be
                        included by default because it is an expensive operation, and may take some additional time.


Finally, so you can link where SAAVs are to function, output the assigned gene functions to a file and the fastas for each gene
```bash
anvi-export-functions -c CONTIGS.db -o cog_functions.txt --annotation-sources COG14_CATEGORY,COG14_FUNCTION
anvi-get-sequences-for-gene-calls -c contigs.db -o genes_nt.fa
```


## 📝 Assignment due next class on Canvas
1. Do you find genes that are present in C-section, not present in vaginal birth samples, or vis versa? (Use the gene_detection file and additional layer file)
2. Give me the top three genes with the most AA variation? Normalize by gene length because longer genes have more sites, so raw counts SAAVs will look bigger just due to length, not biology. Normalizing by length lets you compare apples to apples. (Use the snvs.txt file)
3. What are these genes and what functions are involved in? (Use fasta file and cog_functions.txt) 
