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

Log in on LEAP2, go to your `microbial_genomics` directory and create a working directory:

```bash
cd </path/to/microbial_genomics>
mkdir pangenomes
cd pangenomes
```

Download and extract the fasta files for the *V. jasicida* genomes:

```bash
curl -L https://cloud.uol.de/public.php/dav/files/LdgMQW6ixzzPKzS -o V_jascida_genomes.tar.gz
tar -xzvf V_jascida_genomes.tar.gz
mv V_jascida_genomes V_jasicida_genomes
cd V_jasicida_genomes
ls
```

You might notice that, in addition to the MGL genomes, we have a file for a reference genome. This is a dataset downloaded from the [NCBI RefSeq database](https://www.ncbi.nlm.nih.gov/refseq); it contains the complete genome sequence of *Vibrio jasicida* strain 090810c (accession: GCF_002887615.1).

## Preparing Input for Anvi'o

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
                              -o V_jasicida_${g}.db \
                              --num-threads 4 \
                              -n V_jasicida_${g}
done
```

## Annotating Contig Databases

Anvi-o contig databases i, in addition to containing the genome sequences, can hold a lot of additional information per genome, such as gene predictions, *k*-mer frequencies, gene functions, and so on. Like in [CL6: Genome Annotation](./CL6.md), we can use several Anvi'o programs, such as to identify bacterial single-copy core genes, ribosomal RNAs, transfer RNAs, and annotate the genes with functions in each database:

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

Let's take a quick look at the completeness of the genomes:

```bash
anvi-estimate-genome-completeness -e external-genomes.txt
```

Could you tell what genome is more likely to have some contamination?

## Computing the Pangenome

The first step of computing a pangenome is to generate a genomes storage file, which essentially merges all the contigs databases into a single file:

```bash
anvi-gen-genomes-storage -e external-genomes.txt -o V_jasicida-GENOMES.db
```

We can then compute the pangenome:

```bash
anvi-pan-genome -g V_jasicida-GENOMES.db --project-name V_jasicida --num-threads 4
```

This program organizes genes found within the genomes storage database to create a pangenome database. It performs three major things:

1. Calculates the similarity between all gene amino acid sequences (all-vs-all) found in genomes described in the genomes storage database using DIAMOND. Although there are some options:
     * NCBI’s BLASTp can be used instead of DIAMOND specifying the `--use-ncbi-blast flag`, but this is 10-1000x slower.
     * You can focus on a subset using the `--genome-names parameter`, rather than analyzing the full genomes.
     * Excluding partial genes from the analysis using the flag `--exclude-partial-gene-calls`.
     
2. Resolves gene clusters using the DIAMOND/BLAST results using the MCL (Markov Cluster) algorithm after discarding weak hits from the search results.

3. Performs additional analyses of gene clusters for downstream analyses and visualization tasks. These analyses include,
    * Multiple sequence alignment of amino acid sequences in each gene cluster.
    * Computation of functional and geometric homogeneity indices (i.e., how similar are the annotations of genes within a cluster?).
    * Computation of average amino-acid identity (AAI) within each gene cluster.
    * Hierarchical clustering analysis of gene clusters based on their distribution across genomes, and of genomes based on their sharing of the gene pool.


## Evaluating the Pangenome

The pangenome tells us about the similarities and dissimilarities between the included genomes given the amino acid sequences of open reading frames we identified within each one of them. However, we can also compare genomes to each other by computing the average nucleotide identity (ANI) between them.

```bash
anvi-compute-genome-similarity --external-genomes external-genomes.txt \
                               --program pyANI \
                               --output-dir ANI \
                               --num-threads 4 \
                               --pan-db V_jasicida/V_jasicida-PAN.db
```

Summarize the main features of the pangenome:

```bash
anvi-script-add-default-collection -p V_jascida/V_jascida-PAN.db 
anvi-summarize -p V_jasicida/V_jasicida-PAN.db -g V_jasicida-GENOMES.db -C DEFAULT
```

Download the `SUMMARY` directory to your local computer and open the `index.html` file in a web browser to inspect details of the generated pangenome.

This is the end of CL8!
