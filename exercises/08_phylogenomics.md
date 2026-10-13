# CL14: Phylogenomics

In this tutorial, we will put our genomes in a phylogenomic context to elucidate the evolutionary relationships among each other. This tutorial is based on the [Happy Belly Bioinformatics](https://astrobiomike.github.io)' [Introduction to Phylogenomics tutorial](https://astrobiomike.github.io/genomics/phylogenomics).

## 🧠 Learning Objectives

By the end of this tutorial, you should be able to:
* Explain how single-copy core genes are identified and used in phylogenomics.
* Interpret phylogenetic trees in the context of evolutionary relationships.
* Recognize limitations and sources of bias in genome-based phylogenetic inferece.

## Pipeline Overview

For this tutorial, we will follow one of the most common phylogenomics pipelines:

1. Identify target gene from genomes of interest
2. Align each set of genes
3. Concatenate the alignments
4. Reconstruct the phylogenetic tree

<img width="500" height="1275" alt="image" src="https://astrobiomike.github.io/images/phylo-img.png" />

To do this, we will be using [GToTree](https://doi.org/10.1093/bioinformatics/btz188), a user-friendly phylogenomics workflow.

<img width="950" height="573" alt="image" src="https://astrobiomike.github.io/images/GToTree-Overview.png" />

## Setting Up Working Directory and Environment

Login onto LEAP2 and create a working directory in your `microbial_genomics` directory:

```bash
cd <path/to/microbial_genomics>
mkdir phylogenomics
cd phylogenomics
```

Install GToTree:

```bash
conda create -y -n gtotree -c conda-forge -c bioconda -c defaults -c astrobiomike gtotree
conda activate gtotree
GToTree -h
```

## Data
*Synechococcus* is one of the abundant cyanobacterium found throughout the ocean and a major component of the phytoplankton. Years ago, a project generated genome sequence data for several marine *Synechococcus*. One of the goals of this project was to place these "new *Synechococcus*" in the evolutionary landscape of the known *Synechococcus*. A taxonomy assignment tool would tell indeed classify these genomes as belonging to *Synechococcus*; a necessary first step. However, the project aimed to identify which *Synechococcus* the samples were more closely related to. This is where a *de novo* phylogenomic tree comes in handy.'

Download the tutorial data:

```bash
curl -L -o syn-gtotree-example.tar.gz https://ndownloader.figshare.com/files/23629763
tar -xvzf syn-gtotree-example.tar.gz
mv syn-gtotree-example data
cd data
```

Look at the `ref-syn-accs.txt` file:

```bash
wc -l ref-syn-accs.txt
head ref-syn-accs.txt
```

There are 20 NCBI assembly accessions in there. These include 19 marine *Synechococcus* references, and one for *Gloeobacter violaceus* (a more distant cyanobacterium) that we can use as an outgroup to root our tree.

We have 12 our of newly generated genomes in fasta files:

```bash
ls *.fa | wc -l
```

Create a file that lists these files to use as input for GToTree:

```bash
ls *.fa > our-genome-fasta-files.txt
head our-genome-fasta-files.txt
cd ..
```

Since we are working with *Synechococcus*, which are Cyanobacteria, we can use the Cyanobacteria Hidden Markov Model (HMM) set for Cyanobacteria included with GToTree that holds 251 single-copy genes:

```bash
gtt-hmms
conda deactivate
```

> [!NOTE]
> Why are we focusing on single-copy genes shared across *Synechococcus*?

## Running GToTree

Here we’ll start the command, and then breakdown the code and what GToTree is doing while it’s running. As run here, the largest time spent is by far in making the alignments (running with more threads means more alignments can run concurrently, which leads to a large improvement in runtime). With aligning and treeing the full 251 genes from the Cyanobacteria HMM set, this took ~6 minutes on my laptop and 25 minutes on the Binder. If we’d like this to run much faster just for this example’s sake, we can just target the 15 genes in the Universal HMM set, which then takes < 1 minute on my laptop and 6 minutes on the binder (though we may be sacrificing some ability to resolve their relationships by using a much smaller gene set than is ideal for our target breadth of diversity).There are 2 required things we need to tell GToTree: 1) which genomes we want to work on; and 2) which target genes we want to use to build our phylogenomic tree.

Start an interactive shell:

```bash
sinteractive -p shared -n 4 --mem-per-cpu=10G --time=2:00:00
```

Run GToTree:

```bash
conda activate gtotree
GToTree -a data/ref-syn-accs.txt -f data/our-genome-fasta-files.txt -H Cyanobacteria -o output -t -L Species -n 4  -j 4
```

Code breakdown:

- `-a`: file containing the input genomes we are providing as NCBI assembly accessions
- `-f`: file containing the input genomes we are providing as fasta files
- `-H`: the target genes we want to search for
- `-o`: set the name of the output directory
- `-t`: add taxonony information to the labels of the reference genomes
- `-L`: specify the taxonomy level we want in our labels. “Species” actually includes genus, species, and strain designations from NCBI
- `-n`: sets the number of CPUs to use
- `-j`: sets the number of jobs to run in parallel where possible (e.g.m download and search 4 genomes at a time, and run 4 alignments at a time)



`Bacteria_71` is a collection that anvi’o developers curated by taking Mike Lee’s bacterial SCGs collection, first released in GToTree, which is another easy-to-use phylogenomics workflow.
Let’s see what genes do we have in `Bacteria_71`:
```
anvi-get-sequences-for-hmm-hits -c CONTIGS.db \
                                -p PROFILE.db \
                                -o seqs-for-phylogenomics.fa \
                                --hmm-source Bacteria_71 \
                                --list-available-gene-names
```

For the sake of this simple example, let’s assume we want to use a bunch of ribosomal genes for our phylogenomic analysis: Ribosomal_L1, Ribosomal_L2, Ribosomal_L3, Ribosomal_L4, Ribosomal_L5, Ribosomal_L6.
The following command will give us all these genes from all bins described in the collection `default`:
```
anvi-get-sequences-for-hmm-hits -c CONTIGS.db \
                                -p PROFILE.db \
                                -o seqs-for-phylogenomics.fa \
                                --hmm-source Bacteria_71 \
                                -C default \
                                --gene-names Ribosomal_L1,Ribosomal_L2,Ribosomal_L3,Ribosomal_L4,Ribosomal_L5,Ribosomal_L6
```

Let's view the output:
```
less seqs-for-phylogenomics.fa
```
Press `q` to exit out of `less`. 


You can see this is not an alignment file but we need a single concatenated alignment of gene sequences per genome. 

Actually, the same function `anvi-get-sequences-for-hmm-hits` has a flag, `--concatenate-genes` to get your genes of interest to be concatenated. 
Let’s do that:
```
anvi-get-sequences-for-hmm-hits -c CONTIGS.db \
                                -p PROFILE.db \
                                -o seqs-for-phylogenomics.fa \
                                --hmm-source Bacteria_71 \
                                -C default \
                                --gene-names Ribosomal_L1,Ribosomal_L2,Ribosomal_L3,Ribosomal_L4,Ribosomal_L5,Ribosomal_L6 \
                                --concatenate-genes

```

Now, you are going to get an error that we need to use the flag `--return-best-hit`. The reason being that even in the most complete genomes, there may be multiple HMM hits for a given SCG.
As a solution to this problem, anvi’o asks you to use the --return-best-hit flag, which will return the most significant HMM hit if there are more than one gene that matches to the HMM of a given gene in a given genome. Fine. 
Let’s do that, then:
```
anvi-get-sequences-for-hmm-hits -c CONTIGS.db \
                                -p PROFILE.db \
                                -o seqs-for-phylogenomics.fa \
                                --hmm-source Bacteria_71 \
                                -C default \
                                --gene-names Ribosomal_L1,Ribosomal_L2,Ribosomal_L3,Ribosomal_L4,Ribosomal_L5,Ribosomal_L6 \
                                --concatenate-genes \
                                --return-best-hit
```

Let's look at the output again
```
less seqs-for-phylogenomics.fa
```

We are getting somewhere, but not quite there yet. See, the output is in DNA alphabet, which may not be the best option for phylogenomic analyses, especially if the genomes you have are coming from distant clades (which happens to be the case for IGD). 
Fortunately, you can easily switch to AA alphabet with an additional flag `--get-aa-sequences`. Ok, one last time:
```
anvi-get-sequences-for-hmm-hits -c CONTIGS.db \
                                -p PROFILE.db \
                                -o seqs-for-phylogenomics.fa \
                                --hmm-source Bacteria_71 \
                                -C default \
                                --gene-names Ribosomal_L1,Ribosomal_L2,Ribosomal_L3,Ribosomal_L4,Ribosomal_L5,Ribosomal_L6 \
                                --concatenate-genes \
                                --return-best-hit \
                                --get-aa-sequences
```
```
less seqs-for-phylogenomics.fa
```
Congrats, we did it. Moving on. 

## 🧪 Step 4: Compute a phylogenomic tree

Once you have your concatenated genes, which you now have them in `seqs-for-phylogenomics.fa`, it is time to perform the phylogenomic analysis.
Here we will use the program `anvi-gen-phylogenomic-tree`, which accepts a FASTA file and uses FastTree to infer phylogenetic trees.
```
anvi-gen-phylogenomic-tree -f seqs-for-phylogenomics.fa \
                           -o phylogenomic-tree.txt
```

Visualize the tree like so:
```
anvi-interactive --tree phylogenomic-tree.txt \
                 -p temp-profile.db \
                 --title "Pylogenomics of IGD Bins" \
                 --manual
```

Now, we can see how related our bins are. For instance, we can see that the S. hominis, S. epidermidis, and S. aureus, all Staphylococci, are correctly placed next to one another. 
The remaining bins are different organisms, placed appropriately from one another. 


## 🧪 Step 5: Phylogenomics for closely related organisms

What makes ribosomal SCGs powerful for some applications of phylogenomics, makes them weak for other applications. Comparing very similar genomes (pangenomics) may require the inclusion of a larger number of genes that occur in all genomes of interest to increase the signal of divergence. 

Let's test this with our_ E. faecalis_ and _E. faecium_ genomes from last week. We will first grab the same ribosomal SCGs like before:

```
anvi-get-sequences-for-hmm-hits --external-genomes additional-files/pangenomics/external-genomes.txt \
                                -o concatenated-proteins.fa \
                                --hmm-source Bacteria_71 \
                                --gene-names Ribosomal_L1,Ribosomal_L2,Ribosomal_L3,Ribosomal_L4,Ribosomal_L5,Ribosomal_L6 \
                                --concatenate-genes \
                                --return-best-hit \
                                --get-aa-sequences
```

Compute the tree and visualize it
```
anvi-gen-phylogenomic-tree -f concatenated-proteins.fa \
                           -o phylogenomic-tree.txt
```
```
anvi-interactive -p phylogenomic-profile.db \
                 -t phylogenomic-tree.txt \
                 --title "Enterococcus Tree" \
                 --manual
```

As you can see, many of the genomes have a flat line in the phylogenomic tree, which indicates that they have an identical set of the ribosomal proteins we used. This could often be the case for very closely related populations you want to study. 

Let's use the pangenomic analysis from last week. Select ~50 genes randomly, and store them in a new collection called "Some_SCGs" to generate a phylogenomic tree using those:
```
anvi-display-pan -g Enterococcus-GENOMES.db \
                 -p PAN/Enterococcus-PAN.db \
                 --title "Enterococccus Pan"
```

The program anvi-get-sequences-for-gene-clusters is what we will use to export alignments for genes in our gene clusters. Here, we declare the collection name and the bin id in our anvi’o pan database, and export sequences:
```
anvi-get-sequences-for-gene-clusters -g Enterococcus-GENOMES.db \
                                     -p PAN/Enterococcus-PAN.db \
                                     --collection-name default \
                                     --bin-id Some_SCGs \
                                     --concatenate-gene-clusters \
                                     -o scgs-concatenated-proteins.fa \
                                     --max-num-genes-from-each-genome 1
```

Generate the new tree using the new SCGs
```
anvi-gen-phylogenomic-tree -f scgs-concatenated-proteins.fa \
                           -o scgs-phylogenomic-tree.txt
```

View the tree now:
```
anvi-interactive -p phylogenomic-profile.db \
                 -t scgs-phylogenomic-tree.txt \
                 --title "Enterococcus Tree" \
                 --manual

```

## 📝 No assignment this week :)
