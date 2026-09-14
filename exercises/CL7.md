# CL7: Ortholog Analysis

This tutorial is based on [tutorials from the OrthoFinder's developers](https://davidemms.github.io/menu/tutorials.html). The purpose is getting familiar with ortholog analysis, a classic analysis in comparative genomics.

---
## 🧠 Learning Objectives

By the end of this exercise, you should be able to:
- Run analysis of gene orthologs
- Interpret different types of outputs

## 🧑🏻‍💻 Installing OrthoFinder

[OrthoFinder](https://github.com/davidemms/OrthoFinder) is an easy-to-use, quick and comprehensive tool for comparative genomics. It detects orthologs based on sequence similarity, creates orthogroups (proxies of gene families) based on a clustering algorithm, infers rooted gene trees for all orthogroups, and identifies putative gene duplication events. It also infers a rooted species tree for the organisms being analyzed and maps the gene duplication events to branches in the species tree. The only input OrthoFinder needs is a set of protein sequence files (one per species) in FASTA format.

Log into LEAP2 and create a working directory:

```bash
cd </path/to/>microbial_genomics
mkdir orthofinder
cd orthofinder
```

Install OrthoFinder using Conda:

```bash
conda create -n orthofinder python=3.12
conda activate orthofinder
conda install orthofinder
```

Check installation by printing the help message:

```bash
orthofinder -h
```

## 📈 Protein data

We are going to run a phylogenomic analysis across a set of model species: mouse, human, frog, zebrafish, Japanese puffer (Takifugu rubripes) and fruit fly (Drosophila melanogaster). Keep in mind that, while these organisms are not microbes, this type of analysis can easily be transferred to any kind of organism.

Inside your `orthofinder` directory, create a directory called “data”, go in there and obtain the path to that directory:

```bash
mkdir data
cd data/
pwd
```

Go to https://www.ensembl.org/, this is generally the first place to look for proteomes. Click on “Human” under “Favourite genomes”. (If you’re downloading data from other websites you might find this post useful: Getting OrthoFinder input data)

OrthoFinder requires as input the amino acid sequences for all the protein coding genes in your species of interest. The sequences for each species should be in a separate file with filename extension “.fa”, “.faa”, “.fasta”, “.fas” or “.pep”. When a genome of a species is sequenced and made available, two major steps are performed, assembly and annotation. Assembly is the piecing together of the individual reads into the genome sequence. Annotation is the identification of features of interest in the genome assembly, such as protein coding genes. Therefore, the files we need will often be in a section called ‘annotation’. On Ensembl, on the right hand side, under “Gene annotation” click “Download FASTA”.

