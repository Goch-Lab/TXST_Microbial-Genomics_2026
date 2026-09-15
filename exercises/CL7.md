# CL7: Ortholog Analysis

This tutorial is based on [tutorials from the OrthoFinder's developers](https://davidemms.github.io/menu/tutorials.html). The purpose is getting familiar with ortholog analysis, a classic analysis in comparative genomics.

---
## 🧠 Learning Objectives

By the end of this exercise, you should be able to:
- Run analysis of gene orthologs
- Interpret different types of outputs

## 🧑🏻‍💻 Installing OrthoFinder

[OrthoFinder](https://github.com/davidemms/OrthoFinder) is an easy-to-use, quick and comprehensive tool for comparative genomics. It detects orthologs based on sequence similarity, creates orthogroups (proxies of gene families) based on a clustering algorithm, infers rooted gene trees for all orthogroups, and identifies putative gene duplication events. It also infers a rooted species tree for the organisms being analyzed and maps the gene duplication events to branches in the species tree. The only input OrthoFinder needs is a set of protein sequence files (one per species) in FASTA format.

<img width="7201" height="3022" alt="image" src="https://upload.wikimedia.org/wikipedia/commons/a/ad/OrthoFinderWorkflow.jpg?utm_source=en.wikipedia.org&utm_campaign=index&utm_content=original" />

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

The fasta files were downloaded from [ENSEMBL](https://www.ensembl.org) and may contain multiple isoforms per gene. If we ran OrthoFinder on these raw files it would take ~10x longer than necessary and could lower the accuracy. Use a script provided with OrthoFinder to extract the longest variant per gene on an interactive shell:

```bash
sinteractive -p shared -n 1 --mem-per-cpu=10G --time=1:00:00
conda activate orthofinder
for f in *.faa ; do primary_transcript $f ; done
conda deactivate
```

## 🖥️ Running OrthoFinder

Go back to the `orthofinder` directory:

```bash
cd ..
```

Submit the following SLURM script for OrthoFinder (**TIP**: open a text editor as per usual, decide on a file name, *edit* the script and submit it:

```bash
#!/bin/bash
#SBATCH --job-name=<job name>
#SBATCH --partition=shared
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --time=5:00:00
#SBATCH --mem=40G

# Get started
echo "Job started on $(hostname) at $(date)"

source ~/.bashrc
conda activate orthofinder

#Variables
export PROT=</path/to/primary_transcripts>

#Commands
orthofinder -t $SLURM_NTASKS -f $PROT

# Finish up
conda deactivate

echo "Job Ended at $(date)"
```

Keep track of the job and ensure it completes successfully as we have done in previous sessions. This will take a while.

## 🧑🏻‍💻 Exploring OrthoFinder' results

The analysis OrthoFinder performs is pretty extensive so we will start with the key OrthoFinder results files and explore them as you would explore your own results. You can also see a complete listing of the OrthoFinder results files on their [GitHub page](https://github.com/OrthoFinder/OrthoFinder#output-files).

By default OrthoFinder creates a results directory called `OrthoFinder`. Explore the contents of the results directory: 

```bash
cd data/primary_transcripts/OrthoFinder/Results_<Date>/
ls
```

<ins>General Statistics</ins>

The first thing to check is how many genes were assigned to orthogroups. OrthoFinder should have printed a text like this in the SLURM output:

```bash
OrthoFinder assigned 121743 genes (92.9% of total) to 17981 orthogroups.
```

Otherwise, you can also find this information in the `Comparative_Genomics_Statistics/Statistics_Overall.tsv.` file.

This is pretty good, in general it is nice to see >80% of your genes assigned to orthogroups. Fewer than this means that you are probably missing orthology relationships that actually exist for some of the remaining genes, poor species sampling is the most likely cause for this, although this will depend on the organisms you are studying. Let’s also check the percentages on a per species basis:

```bash
less Comparative_Genomics_Statistics/Statistics_PerSpecies.tsv
```

This is a tab-separated file (“.tsv”), in which columns are delimited by tabs. TSV files like one are best visualized in a spreadsheet (like in Excel). It is up to you if you would like to download it to your local computer to explore it more easily.

You may notice that all vertebrates have >90% of their genes assigned to orthogroups, whereas *Drosophila* has about 76% of its genes assigned. This is probably due to species sampling. The four vertebrate species are relatively closely related, whereas the species sampling around both *Drosophila* was poor.

<ins>Orthogroups</ins>

Often we are interested in group-wise species comparisons, that is comparisons across a clade of species rather than between a pair of species. The generalization of orthology to multiple species is the orthogroup. Just like orthologs are the genes descended from a single gene in the last common ancestor of a pair of species, **an orthogroup is the set of genes descended from a single gene in a group of species**. So, if we want to do a comparison of the "equivalent" genes in a set of species, we need to do the comparison across the genes in an othogroup. The orthogroups are in the file `Orthogroups.tsv`:

```bash
less Orthogroups/Orthogroups.tsv
```

This table has one orthogroup per line and one spcies per column and is ordered from the largest orthogroup to the smallest.

<ins>Species Tree</ins>

Let’s look at the species tree next:

```bash
less Species_Tree/SpeciesTree_rooted.txt
```

This file is in the Newick format. The Newick format is a text-based way to represent phylogenetic trees using parentheses and commas:

 - Parentheses group related nodes or sister taxa together.
 - Commas separate items within the same group.
 - Names identify individual leaf nodes (like species).
 - Colons optionally add branch lengths or node support values.
 - Semicolons mark the end of the entire tree string.

<img width="3847" height="3466" alt="image" src="https://josephcrispell.github.io/assets/img/blog/newick/thumbnail.svg" />

There are several programs and online tools that can be used to visualize trees in the Newick format. Download the species tree file to your local computer and upload it to the [ETE Toolkit tree viewer](https://etetoolkit.org/treeview).

This tree has been inferred by OrthoFinder using the [STAG](https://doi.org/10.1101/267914) algorithm and rooted using the [STRIDE](https://doi.org/10.1093/molbev/msx259) algorithm, so it is ready to interpret (ordinarily you would have to root a tree yourself first). You can see here that *Drosophila* is on longer branches than the other species, as mentioned above. If you know what the species tree should look like, you should check that the tree matches what you expect. The tree OrthoFinder inferred here is correct. If the species tree is not correct then this will not impact the orthogroup inference, but it might affect the other inferences, such as some gene duplication events.

This is the end of CL7!
