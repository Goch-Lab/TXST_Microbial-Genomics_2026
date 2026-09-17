# CL8: Pangenomics

In this tutorial, we will run a pangenomic analysis using [Anvi'o](https://anvio.org/). This tutorial is large based on [a workshop run by the Anvi'o developer](https://merenlab.org/tutorials/vibrio-jasicida-pangenome/).

## 🧠 Learning Objectives

By the end of this exercise, you should be able to:
* Build a pangenome from multiple genomic datasets.

## 🧬 Genome Data

This tutorial uses genome data generated for seven isolates of *Vibrio jasicida* during the Microbial Diversity course at the Marine Biological Laboratory (MBL) in 2018. The isolates were obtained from various places in Woods Hole, Massachusetts, including: the Eel Pond, the Marine Resources Center, and the Great Harbor:

| Plate Number | Researcher         | Source                         |
|--------------|--------------------|--------------------------------|
| 12           | Peggy Lai          | Seawater, Great Harbor         |
| 13           | Peggy Lai          | Seawater, Great Harbor         |
| 14           | Peggy Lai          | Seawater, Great Harbor         |
| 47           | Danielle Campbell  | Seawater, Eel Pond             |
| 52           | Brittni Bertolet   | Unknown                        |
| 53           | Sarah Schwenck     | Coral, Marine Resources Center |
| 55           | Monica E. McCallum | Coral, Marine Resources Center |

Download and extract the fasta files for the *V. jasicida* genomes:

```bash
curl -L https://cloud.uol.de/public.php/dav/files/LdgMQW6ixzzPKzS -o V_jascida_genomes.tar.gz
tar -xzvf V_jascida_genomes.tar.gz
cd V_jascida_genomes
ls
```

You might notice that, in addition to the MGL genomes, we have a file for a reference genome. This is a dataset downloaded from the [NCBI RefSeq database](https://www.ncbi.nlm.nih.gov/refseq); it contains the complete genome sequence of *Vibrio jasicida* strain 090810c (accession: GCF_002887615.1).

## Converting Fasta files into Anvi'o Contig Databases

First, we need to convert the fasta files into an Anvi'o contigs database. Start an interactive shell, activate your Anvi'o environment, and create a file that is easier to loop through:

```bash
sinteractive -p shared -n 4 --mem-per-cpu=10G --time=2:00:00
conda activate anvio-9
ls *fasta | awk 'BEGIN{FS="_"}{print $1}' > genomes.txt
cat genomes.txt
```

The fasta files contain genome assemblies from the isolates, which include many short sequences. Therefore, it is a good idea to remove short sequences since they may be coming from low-abundance contaminants that didn’t assemble well and/or influence how gene clusters are formed. Let's go through each entry in our genomes.txt file and use the program anvi-script-reformat-fasta to create copies of each fasta file only with sequences ≥2,500 bp:

```bash
for g in `cat genomes.txt`
do
    echo
    echo "Working on $g ..."
    echo
    anvi-script-reformat-fasta ${g}_scaffolds.fasta \
                               --min-len 2500 \
                               --simplify-names \
                               -o ${g}_scaffolds_2.5K.fasta
done
```

Now we can generate the contig databases:

```bash
for g in `cat genomes.txt`
do
    echo
    echo "Working on $g ..."
    echo
    anvi-gen-contigs-database -f ${g}_scaffolds_2.5K.fasta \
                              -o V_jascida_${g}.db \
                              --num-threads 4 \
                              -n V_jascida_${g}
done
```

## Annotating Contig Databases

Anvi-o contig databases i, in addition to containing the genome sequences, can hold a lot of additional information per genome, such as gene predictions, *k*-mer frequencies, gene functions, and so on. Like in [CL6: Genome Annotation](../CL6.md), we can use several Anvi'o programs, such as to identify bacterial single-copy core genes, ribosomal RNAs, transfer RNAs, and annotate the genes with functions in each database:

```bash
for g in *.db
do
    anvi-run-hmms -c $g --num-threads 4
    anvi-run-ncbi-cogs -c $g --num-threads 4
    anvi-scan-trnas -c $g --num-threads 4
    anvi-run-scg-taxonomy -c $g --num-threads 4
done
```

Take a look at some features of the isolate genomes:

```bash
anvi-display-contigs-stats *.db --report-as-text -o contigs_stats.txt
cat contigs_stats.txt
```

When working with a bunch of contig databases in Anvi'o, a  file called `external-genomes` is describing the dataset is required for most tools:

```bash
anvi-script-gen-genomes-file --input-dir . -o external-genomes.txt
```

The structure of the external-genomes file is simple: it is a two-column TAB-delimited file. One can easily create it using EXCEL or any kind of text editor. But anvi’o comes with a script, anvi-script-gen-genomes-file to create one from all the contigs databases found in a directory. So let’s use that to generate an external genomes file:
So, this program implements pangenomics and organizes genes found within a genomes-storage-db to create a pan-db.
It performs three major things for its user:

1. Calculates the similarity between all gene amino acid sequences (all-vs-all) found in genomes described in your genomes-storage-db using DIAMOND. Although you have some options:
     * You can use the NCBI’s BLAST program blastp instead of DIAMOND using the --use-ncbi-blast flag, but this is 10-1000x slower.
     * Instead of analyzing all genomes, you can focus on a subset using the `--genome-names parameter`.
     * exclude genes that are partial from your analysis using the flag `--exclude-partial-gene-calls` if you think you must.
     
3. Resolves gene clusters using the DIAMOND/BLAST results via the MCL (Markov Cluster) algorithm after discarding weak hits from the search results using the `--minbit` heuristic.

4. Performs additional analyses of gene clusters for downstream analyses and visualization tasks. These analyses include,
    * Multiple sequence alignment of amino acid sequences in each gene cluster
    * Computation of functional and geometric homogeneity indices (i.e. how similar the annotations of genes within a cluster?)
    * Computation of average amino-acid identity (AAI) within each gene cluster
    * Hierarchical clustering analysis of gene clusters based on their distribution across genomes, and genomes based on their sharing of the gene pool


Now the pangenome is ready to display. This is how you can run it:
```
anvi-display-pan -g Enterococcus-GENOMES.db \
                 -p PAN/Enterococcus-PAN.db \
                 --title "Enterococccus Pan"
```

I am going to show you how to clean this view up and change colors.
At the end, make sure to "Save State".

Now not only can we see how our _E. faecalis_SHARON MAG looks like compared to available genomes, but we can also see that it is not missing or carrying a great number of proteins compared to other genomes. The clustering of genomes based on gene clusters indicates that it is most similar to the genome of _Enterococcus faecalis_ 6512. 

## 🧪 Step 3: Organizing gene clusters based on a forced synteny
By default, anvi’o will organize gene clusters in a given pangenome based on their distribution patterns across genomes. Which, by definition, will break gene synteny, i.e., two gene clusters that are next to each other in the display will not necessarily carry genes that are next to each other in respective genomes. If you have a relatively complete genome, you can ask anvi’o to organize gene clusters based on the organization of genes in that genome.

Here, for instance, we can force the organization of gene clusters based on the synteny of genes in our _E. faecalis_ genome:
<img width="1039" height="1129" alt="image" src="https://github.com/user-attachments/assets/919615f4-bffa-4a90-a8ec-ae6eeecb11e7" />


## 🧪 Step 4: Average nucleotide identity (ANI)

The pangenome tells us about the similarities and dissimilarities between those genomes given the amino acid sequences of open reading frames we identified within each one of them. But we could also compare genomes to each other by computing the average nucleotide identity between them. Anvi’o comes with a program to do that, which uses PyANI by Pritchard et al. to pairwise align DNA sequences between genomes to estimate similarities between them. 

We can compute average nucleotide identity among our genomes, and add them to the pan database. This can take a bit of time, especially as you add more genomes. 

I came across an error when running this initially, to fix it, we need to downgrade the version of a package installed in this conda environment, like so:
```
pip install matplotlib==3.5.1
```

Now you can run the ANI analysis:
```
anvi-compute-genome-similarity -e additional-files/pangenomics/external-genomes.txt \
                               -i additional-files/pangenomics/internal-genomes.txt \
                               --program pyANI \
                               -o ANI \
                               -T 4 \
                               --pan-db PAN/Enterococcus-PAN.db
```

Once it's done, visualize it again. 
```
anvi-display-pan -g Enterococcus-GENOMES.db \
                 -p PAN/Enterococcus-PAN.db \
                 --title "Enterococccus Pan"
```

And take a careful look at your ‘Layers’ tab so you can visualize your pangenome with the new ANI results.


## 🧪 Step 5: Functional enrichment analyses
One of the things we are often interested in is this question: which functions are associated with a particular organization of genomes due to their phylogenomic or pangenomic characteristics?

For instance, genomes in this pangenome are organized into two distinct groups based on differential distribution of gene clusters. That is of course not surprising, since these genomes are classified into two distinct ‘species’ within the genus Enterococcus. So you can certainly imagine more appropriate or interesting examples where you may be wondering about _**functional enrichment**_ across groups of genomes that do not have such clear distinctions at the level of taxonomy.

This is done with `anvi-compute-functional-enrichment-in-pan`.

This program utilizes information you provide by splitting your genomes into multiple groups and finds functions that are characteristic to any of those groups, i.e., functions that are enriched in genomes that belong to one group, but predominantly absent in genomes outside of it.

It simply determines that for every function, the associated groups are the ones in which the occurrence of the function of genomes is greater than the expected occurrence under a uniformal distribution (i.e. if the function was equally probable to occur in genomes from all groups). 

First, you need to define how you would like to group your genomes in the pangenome so anvi’o can find out which functions are characteristic of each group. We'll group by clade _faecalis_ and _faecium_.

```
anvi-import-misc-data -p PAN/Enterococcus-PAN.db \
                      --target-data-table layers \
                      additional-files/pangenomics/additional-layers-data.txt
```

Once that info is imported, we can re-run the pangenome, where you will see the new clade identifiers. 
```
anvi-display-pan -g Enterococcus-GENOMES.db \
                 -p PAN/Enterococcus-PAN.db \
                 --title "Enterococccus Pan"
```

Exit out. 

Now we can use the program anvi-compute-functional-enrichment-in-pan to ask anvi’o to identify and report functions that are enriched in either of these clades, along with the gene clusters they are associated with:
```
anvi-compute-functional-enrichment-in-pan -p PAN/Enterococcus-PAN.db \
                                          -g Enterococcus-GENOMES.db \
                                          --category-variable clade \
                                          --annotation-source COG20_FUNCTION \
                                          -o functional-enrichment.txt
```

This will generate a new file, `functional-enrichment.txt`. Take a look. 

The following describes each column in this output:

* COG20_FUNCTION this column has the name of the specific function for which enrichment was calculated. In this example we chose to use COG20_FUNCTION for functional annotation. You can specify whichever functional annotation source you have in your PAN database using the `--annotation-source`, and then the analysis would be done according to that annotation source. Even if you have multiple functional annotation sources in your genome storage, only one source could be used for a single run of this program. If you wish, you could run it multiple times and each time use a different annotation source. If you don’t remember which annotation sources are available in your genomes storage, you can use the flag --list-annotation-sources.

* enrichment_score is a score to measure how much this function is unique to the genomes that belong to a specific group vs. all other genomes in your pangenome. This score was developed by Amy Willis. For more details about how we generate this score, see the note from Amy below.

* unadjusted_p_value is the p-value for the enrichment test (unadjusted for multiple tests).

* adjusted_q_value is the adjusted q-value to control for False Detection Rate (FDR) due to multiple tests (this is necessary since we are running the enrichment test each function separately).

* associated_groups is the list of groups (or labels) in your categorical data that are associated with the function. Notice that if the enrichment score is low, then this is meaningless (if the function is not enriched, then it is not really associated with any specific group/s).
  
* function_accession is the function accession number.

* gene_clusters_ids are the gene clusters that were associated with this function. Notice that each gene cluster would be associated with a single function, but a function could be associated with multiple gene clusters.

* p_faecium, p_faecalis for each group there will be a column with the portion of the group members in which we detected the function.
  
* N_faecium, N_faecalis for each group these columns specify the total number of genomes in the group.

>[!NOTE]
>A note from Amy Willis regarding the functional enrichment score
>
>The functional enrichment score proposed here and implemented in anvi’o is the Rao test statistic for equality of proportions. Essentially, it treats each category (here, high light and low light) as an explanatory variable in a logistic regression (binomial GLM), and tests the significance of the categorical variable in explaining the occurrence of the function.
>
>The test accounts for the fact that there may be more genomes observed from one category than the other. As usual, having more genomes makes the test more reliable. There are lots of different ways to do this test, but we did some investigations and found that the Rao test had the highest power out of all tests that control Type 1 error rate. Yay!
>
>Since many users will be looking at testing for enrichment across many functions, by default we adjust for multiple testing by controlling the false discovery rate. For this reason, please report q-values instead of p-values in your paper if you use this statistic.
>

## 🧪 Step 6: Binning gene clusters
Like how we binned metagenomes, we can bin gene clusters based on their distributions across genomes and then summarize the information.
Go ahead and make your gene cluster bins for: CORE ALL, CORE FAECALIS, CORE FAECIUM
Store bins in default. 

To summarize, exit and go back to the command line to run.
```
anvi-summarize -p PAN/Enterococcus-PAN.db \
               -g Enterococcus-GENOMES.db \
               -C default \
               -o PAN_SUMMARY
```
```
open PAN_SUMMARY/index.html
```

This file will link each gene from each genome with every selection you’ve made and named through the interface and will also give you access to the amino acid sequence and function of each gene.

## 🧪 Optional: Calculating open vs closed pangenome 
Anvi'o has a function to use rarefaction curve analysis to calculate if a pangenome is open or close, but its only available for the dev version of Anvi'o, not the one we have installed.
I encourage you to later make a new anvio conda environment for the dev version and test this if relevant to your project
This is how we would have done it today. 
```
anvi-compute-rarefaction-curves -p PAN/Enterococcus-PAN.db \
                                --iterations 100 \
                                -o pangenomes.png

```
Example output:
<img width="1000" height="600" alt="image" src="https://github.com/user-attachments/assets/346e7f3f-c631-4b53-8994-d6ed0132f9ba" />


## 📝 Assignment due next class on Canvas
From the `Enterococcus_gene_clusters_summary.txt` file, identify one example of a gene cluster that is present in the core genome of _E. faecalis_ but absent from the core of _E. faecium_. 
Write a short (≈200–300 words) response addressing the following:
1. Identify the gene cluster
   * Report the gene cluster ID and its annotated function (from the function or annotation column).

2. Describe the function
  * What is the putative role of this gene or operon (e.g., transport, metabolism, stress response, virulence)?
  * Include any known or likely biological importance of that function.

3. Discuss potential ecological or evolutionary implications
   * Why might this function be conserved in one species’ core but not the other?
   * How might it reflect differences in habitat, host association, or adaptation strategies between E. faecalis and E. faecium?

