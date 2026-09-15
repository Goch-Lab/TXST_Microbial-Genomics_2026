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

We are going to run a phylogenomic analysis across a set of model species: human (*Homo sapiens*), mouse (*Mus musculus*), tropical clawed frog (*Xenopus tropicalis*), zebrafish (*Danio rerio*), Japanese pufferfish (*Takifugu rubripes*), and fruit fly (*Drosophila melanogaster*). Keep in mind that, while these organisms are not microbes, this type of analysis can easily be transferred to any kind of organism.

Download the `data.tar.gz` from Canvas to your local computer. Upload the same file to your `orthofinder` directory on LEAP2 as we have done in previous sessions. Inside your `orthofinder` directory, extract the content of the compressed directory:

```bash
tar -xvzf data.tar.gz
rm data.tar.gz
```

Unzip the fasta files and explore their content:

```bash
cd data
gunzip *.faa.gz
head *.faa
```

What type of sequences contain these fasta files? Why do they have a ".faa" extension?

>[!NOTE]
> Fasta files containing proteins can have different extensions, including ".fa", ".faa", ".fasta", or ".pep". Always make sure the type of sequences you are dealing with, and that it is the right type for the tools you are using.


