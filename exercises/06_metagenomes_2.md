# CL12: Metagenomics II: Taxonomic Analyses

In this tutorial, we will explore community profiling (taxonomic assignment) in three different modalities: *taxonomic profiling*, *taxonomic binning*, and *taxonomic annotation*. This tutorial is also based on [metagenomics tutorial](https://github.com/Penn-State-Microbiome-Center/KickStart-Workshop-2026/tree/main/Day3-Shotgun) of the KickStart Workshop from the [Penn State One Health Microbiome Center](https://www.huck.psu.edu/research/centers-institutes/one-health-microbiome-center).

## 🧠 Learning Objectives

By the end of this tutorial, you should be able to:

- Distinguish different methods to taxonomically profile microbial communities from metagenomic data.

## Taxonomic Profiling

Taxonomic profiling is usually done to answer the question: "Which taxa are present in my metagenome and what is their abundance?". Currently, taxonomic profiling is quite inaccurate for short-read sequence data. Here, we will use [MetaPhlAn](https://github.com/biobakery/metaphlan) (<ins>Meta</ins>genomic <ins>Ph</ins>y<ins>l</ins>ogenetic <ins>An</ins>alysis) for taxonomic profiling.

Login into LEAP2 and create a working directory:

```bash
cd <path/to/microbial_genomics>
mkdir taxonomic_profiling
cd taxonomic_profiling
```

Install MetaPhlAn:

```bash
sinteractive -p shared -n 4 --mem-per-cpu=10G --time=2:00:00
conda create -n metaphlan3 -c bioconda metaphlan=3.1.0
conda activate metaphlan3
conda install setuptools
metaphlan -h
```

Install the MetaPhlAn database (this is going to take a while):

```bash
mkdir database
scp -r vgz25@leap2.txstate.edu:/mmfs1/home/vgz25/microbial_genomics/taxonomic_profiling/database .
```

Ask the instructor to provide their password. Alternatively, you can try the following command, but it will take a long while:

```bash
metaphlan --install --index mpa_v30_CHOCOPhlAn_201901 --bowtie2db database
```

Download the data:

```bash
mkdir data
cd data
wget -i https://raw.githubusercontent.com/Penn-State-Microbiome-Center/KickStart-Workshop-2022/main/Day5-Shotgun/Data/file_list.txt
gunzip *.gz
cd ..
```

The basic steps of MetaPhlAn are:

<img width="444" height="300" alt="image" src="https://github.com/biobakery/biobakery/blob/master/images/2526749054-MetaPhlAn2.png" />

MetaPhlAn accepts as input short reads from a shotgun metagenomic sequencing experiment in several formats (`.fasta`, `.fastq`, `.bowtie2out` and `.sam`) and outputs the list of detected microbes and their relative abundances. Profile a metagenome from raw reads:

```bash
mkdir output
metaphlan data/SRS014476-Supragingival_plaque.fasta --input_type fasta --force --bowtie2db database --index mpa_v30_CHOCOPhlAn_201901 --bowtie2out output/SRS014476-Supragingival_plaque.fasta.bowtie2out.txt -o output/SRS014476-Supragingival_plaque_profile.txt --nproc 4
conda deactivate
```

This will create two output files:

- `SRS014476-Supragingival_plaque.fasta.bowtie2out.txt`: This file contains the intermediate mapping results to unique gene markers. Alignments are listed one per line in tab-separated columns of read and gene marker.
- `SRS014476-Supragingival_plaque_profile.txt`: This file contains the final computed organism abundances. Organism abundances are listed one clade per line, tab-separated from the clade's percent abundance:
  - The file has a 4-line header:
    - The first line lists the reference marker genes database that MetaPhlAn uses. There are ~1.1M unique clade-specific marker genes identified from ~100k reference genomes (~99,500 bacterial and archaeal and ~500 eukaryotic).
    - The second line lists the path to the tool, the name of the input file and the arguments that were used.
    - The fourth line has the column headers for the columns below.
  - The first column lists clades, ranging from taxonomic kingdoms (Bacteria, Archaea, etc.) to species. The taxonomic level of each clade is prefixed to indicate its level: Kingdom: k__, Phylum: p__, Class: c__, Order: o__, Family: f__, Genus: g__, Species: s__. Let us examine these more clearly by listing them by taxonomic hierarchy:

```bash
grep "s__" -m1 output/SRS014476-Supragingival_plaque_profile.txt | cut -f1 | sed 's/|/\n/g'
```

This `grep` command will look for lines which contain the pattern "s__" that is associated with species and print the first match with the `-m1` argument. This file have 4 tab-separated columns and the taxonomy is listed in the first; so we will use `cut -f1` to extract the first column only (the field at position 1). Finally, the taxonomic levels are separated by the `|` character, which we replace with the new line character `\n`.

The second column lists the corresponding NCBI taxon ID. The third column lists relative abundances. Since sequence-based profiling is relative and does not provide absolute cellular abundance measures, clades are hierarchically summed. Each level will sum to 100%; that is, the sum of all kingdom-level clades is 100%, the sum of all genus-level clades (including unclassified) is also 100%, and so forth. OTU equivalents can be extracted by using only the species-level "s__" clades from this file (again, making sure to include clades unclassified at this level).
Let us check if all orders add up to 100% using `grep`. The orders 'Corynebacteriales' and 'Micrococcales' are in the class 'Actinobacteria'.

```bash
grep o__ output/SRS014476-Supragingival_plaque_profile.txt | grep -v f__
```

Similarly, the families must sum to 100%. In this example, let us also display the fields of interest, i.e., taxonomy names and percentages for ease of viewing:

```bash
grep f__ output/SRS014476-Supragingival_plaque_profile.txt | grep -v g__ | cut -f1,3
```

The fourth column lists additional species for cases where the metagenome profile contains clades that represent multiple species. The species listed in column 1 is the representative species in such cases.

## Taxonomic Binning

We will use [Kraken2](https://github.com/DerrickWood/kraken2) to do taxonomic profiling. In contrast to MetaPhlAn3, which use clade-specific marker genes, Kraken2 uses clade-specific marker *k*-mers. This is summarized in the first figure of the [Kraken1 manuscript](https://genomebiology.biomedcentral.com/articles/10.1186/gb-2014-15-3-r46):

![Kraken](https://user-images.githubusercontent.com/6362936/128582419-7da7594e-9483-4138-859e-d0f0e332bad3.PNG)

In using *k*-mers, Kraken2 has the ability to be blazingly fast, but longer query sequences are required to account for the inherent noise when using *k*-mers in the size ranges that Kraken utilizes. Therefore, **Kraken should not be used for taxonomic profiling**.

Set up working directory and download data:

```bash
cd <path/to/microbial_genomics/metagenomics>
mkdir taxonomic_binning
cd taxonomic_binning
mkdir data
cd data
ln -s ../../assembly/output/default/final.contigs.fa MEGAHIT_default_contigs.fasta
wget -i https://raw.githubusercontent.com/Penn-State-Microbiome-Center/KickStart-Workshop-2021/main/Day5-Shotgun/Data/file_list_fastq.txt
gunzip *.gz
cd ..
```

Install Kraken2:

```
conda create -y -n kraken2 kraken2
conda activate kraken2
kraken2 -h
```

For the purpose of this tutorial, we will be using the smallest Kraken2 database, which includes archaea, bacteria, viruses, plasmids, human sequences, and UniVec_Core (vector contamination). For a real research project, you should use either the default database or one of the other [availabe databases](https://github.com/BenLangmead/aws-indexes/blob/master/docs/k2.md). Even though this database is smallish, it may take a while to download, so you can copy it over from my account:

```bash
scp -r vgz25@leap2.txstate.edu:/mmfs1/home/vgz25/microbial_genomics/taxonomic_binning/k2train8gb .
```

Ask for the instructor's login details. Alternatively, run this command (it will take longer):

```bash
mkdir k2train8gb
cd k2train8gb
wget https://genome-idx.s3.amazonaws.com/kraken/k2_standard_8gb_20210517.tar.gz
tar -xzvf k2_standard_8gb_20210517.tar.gz
cd ..
```

Run Kraken:

```bash
mkdir output
kraken2 kraken2 --db k2train8gb --threads 4 --output output/kraken_default_output.txt --classified-out output/kraken_classified_sequences.fq --use-names --report output/kraken_report.txt data/MEGAHIT_default_contigs.fasta
```

Explore the content of the lines in the `output` directory:
- The `kraken_classified_sequences.fq` files contain fastq formatted files of the contigs with an NCBI taxID in the header of each sequence. This can be helpful for determining the correspondence between contigs and what organisms they originated from.
- The `kraken_default_output.txt` contains a more-compacted version of the one above, but with additional information about what led to Kraken's inference. More details can be found [here](https://github.com/DerrickWood/kraken2/blob/master/docs/MANUAL.markdown#standard-kraken-output-format).
- In the `kraken_report.txt`, the first number is the percent of sequences that covered this taxon, followed by the number assigned to that clade, then to that taxon, a rank code, and then an NCBI taxID. More info can be found [here](https://github.com/DerrickWood/kraken2/blob/master/docs/MANUAL.markdown#sample-report-output-format) about the format.

Take a look at the `kraken_report.txt`file:

```bash
grep -w S output/kraken_report.txt | rev | cut -f1 |rev | sed 's/^ *//g'
```

See how this makes sense considering the BLAST results we saw earlier? In general, *this* is what you want to be using instead of BLAST when you want to identify the taxa of each of your contigs.

# Taxonomic Annotation of Genome Bins

Let's take a first look at the merged profile database for the infant gut dataset metagenome. 
```bash
anvi-interactive -p PROFILE.db -c CONTIGS.db
```

Anvi’o can work with gene-level taxonomic annotations, but gene-level taxonomy is not useful for anything beyond occasional help with manual binning. Once gene-level taxonomy is added into the contigs database, anvi’o will determine the taxonomy of each contig based on the taxonomic affiliation of genes they describe, and display them in the interface whenever possible.

Import that data:
```bash
anvi-import-taxonomy-for-genes -c CONTIGS.db \
                               -p centrifuge \
                               -i additional-files/centrifuge-files/centrifuge_report.tsv \
                               additional-files/centrifuge-files/centrifuge_hits.tsv
```

```
anvi-interactive -p PROFILE.db -c CONTIGS.db
```
You will see an additional layer with taxonomy. In the Layers tab, find the Taxonomy layer, set its height to 200, then drag the layer in between DAY24 and hmms_Ribosomal_RNAs, and click Draw again. Then click the Save State button, and overwrite the default state. This will make sure anvi’o remembers to make the height of that layer 200px the next time you run the interactive interface!


At this point, we don’t have any idea about what genomes we have in this dataset, but anvi’o can make sense of the taxonomic make up of a given metagenome by characterizing taxonomic affiliations of single-copy core genes. 

Take a quick look at taxonomy:
```
anvi-estimate-scg-taxonomy -c CONTIGS.db --metagenome-mode
```

Good, but could be more informative. We can make use of our profile database in the following way, which will give us a little more information about our dataset:
```
anvi-estimate-scg-taxonomy -c CONTIGS.db \
                           -p PROFILE.db \
                           --metagenome-mode \
                           --compute-scg-coverages
```
Looks like information that would have been useful to have in front of us in our interactive interface. Luckily, anvi’o can add these taxonomic insights into a given profile database, if you change the previous command just a bit:
```
anvi-estimate-scg-taxonomy -c CONTIGS.db \
                           -p PROFILE.db \
                           --metagenome-mode \
                           --compute-scg-coverages \
                           --update-profile-db-with-taxonomy
```
```
anvi-interactive -c CONTIGS.db -p PROFILE.db
````
At this point, we have an overall idea about the makeup of this metagenome, but we don’t have any genomes from it. The following sections will cover some of the multiple ways to do this.

## 🧪 Step 3: Manual binning
I'm going to let you try to identify bins on your own first. A few tips:
* Contigs are already clustered together based on tetranucleotide (k=4) frequency.
* Shared/similar coverage of contigs usually indicates contigs that belong together.
* You can increase the inner tree radius (e.g., 5,000) for a better binning experience in the `Main` tab, then save State and Draw. 
* You can select the option show grid in the Main tab (additional settings) for a better demarcation of identified bins.
* If you click on the “Bins” tab at the top left and then select the branch on the tree at the center that holds all the contigs, you will see a real-time estimate of % completion and redundancy.
* You do not have to bin all contigs. Instead, try to identify bins corresponding to an actual genome. 
* Please try to avoid bins with redundancy >10% to ensure they are more accurate and not contaminated.

Example:

<img width="2634" height="1184" alt="image" src="https://github.com/user-attachments/assets/391d9576-5d96-4053-9862-7d57f643f97f" />

Please save your bins as a `collection`. Let's give you collection a name "my_bins". In the anvi’o lingo, a collection is something that describes one or more bins, each of which describes one or more contigs.

Let’s summarize the collection you have just created:
```
anvi-summarize -p PROFILE.db \
               -c CONTIGS.db \
               -C my_bins \
               -o SUMMARY
```
```
open SUMMARY/index.html
```

As you can see from the summary file, at this point bin names are random. It would be more useful to put some order on this front. This becomes an extremely useful strategy, especially when the intention is to merge multiple binning efforts later. For this task we use the program anvi-rename-bins:
```
anvi-rename-bins -p PROFILE.db \
                 -c CONTIGS.db  \
                 --collection-to-read my_bins  \
                 --collection-to-write MAGs  \
                 --call-MAGs \
                 --prefix IGD \
                 --report-file rename-bins-report.txt
```

With those settings, a new collection MAG will be created in which (1) bins with a completion >70% are identified as MAGs (stands for Metagenome-Assembled Genome), and (2) bins and MAGs are attached the prefix IGD and renamed based on the difference between completion and redundancy.

Now we can summarize the new collection:
```
anvi-summarize -p PROFILE.db \
               -c CONTIGS.db \
               -C MAGs \
               -o SUMMARY_AFTER_RENAME

```
```
open SUMMARY_AFTER_RENAME/index.html
```

## 🧪 Step 4: Refining our individual MAGs
To straighten the quality of the MAGs collection, it is possible to visualize individual bins and if needed, refine them. For this we use the program anvi-refine. For instance, if you were to be interested in refining one of the bins in our current collection, you could run this command:
```
anvi-refine -p PROFILE.db \
            -c CONTIGS.db \
            -C MAGs \
            -b IGD_MAG_00001
```
Now the interactive interface only displays contigs from a single bin. During this curation step, one can try different clustering strategies (i.e. by only relying on coverage, or only relying on sequence composition) to identify outliers and investigate carefully whether they may be contaminants. You can select everything and remove those contigs you don’t want to keep in the bin before using the Bins panel to store your updated set of contigs in the database.

Let's refine MAG_00001 together. I want you to do your other bins on your own. 
Storing the new refined bins in the database and it will modify the collection for you, but you will need to re-run `anvi-summarize` if you want the summary output to also be updated.

```
anvi-summarize -p PROFILE.db \
               -c CONTIGS.db \
               -C MAGs \
               -o SUMMARY_AFTER_REFINE
```

## 🧪 Step 5: Manual vs Automatic Binning
Even if you prefer manual binning over automatic binning for the sake of accuracy and control over your data, automatic binning is an unavoidable need due to performance limitations associated with manual binning

The directory additional-files/external-binning-results contains a number of files that describe the binning of contigs in the IGD based on various automatic and manual approaches. These files include (1) outputs from some of the well-known binning algorithms (i.e., GROOPM.txt, MAXBIN.txt, METABAT.txt, BINSANITY_REFINE.txt, MYCC.txt, and CONCOCT.txt), (2) the original binning of this dataset (SHARON_et_al.txt), and the manual binning performed in the anvi’o paper (MEREN_et_al.txt).

You can use the program anvi-show-collections-and-bins to see all collections in your anvi’o profile database:
```
anvi-show-collections-and-bins -p PROFILE.db
```
And use this one to remove one that’s called default, if you have it:
```
anvi-delete-collection -p PROFILE.db \
                       -C default
```
You can create a collection by using the interactive interface (e.g., the default and MAGs collections you just created), or you can import external binning results into your profile database as a collection and see how that collection groups contigs. For instance, let’s import the CONCOCT collection:
```
anvi-import-collection additional-files/external-binning-results/CONCOCT.txt \
                       -c CONTIGS.db \
                       -p PROFILE.db \
                       -C CONCOCT \
                       --contigs-mode

```
```
anvi-show-collections-and-bins -p PROFILE.db
```
OK. Let’s run the interactive interface again with the CONCOCT collection:
```
anvi-interactive -p PROFILE.db \
                 -c CONTIGS.db \
                 --collection-autoload CONCOCT

```
How do your bins compare?

Anvi’o also has a script called anvi-script-merge-collections to merge multiple files from external binning results into a single merged file (don’t ask why):
```
anvi-script-merge-collections -c CONTIGS.db \
                              -i additional-files/external-binning-results/*.txt \
                              -o collections.tsv
```
Visualize all binning results:
```
anvi-interactive -p PROFILE.db \
                 -c CONTIGS.db \
                 -A collections.tsv
```

To visually emphasize relationships between bins, the authors created a state file for us to match colors where bins match
```
anvi-import-state --state additional-files/state-files/state-merged.json \
                  --name default \
                  -p PROFILE.db
```
```
anvi-interactive -p PROFILE.db \
                 -c CONTIGS.db \
                 -A collections.tsv

```
Based on this, would you refine your bins? Probably, but for now let's keep the ones you refined. 


## 📝 Assignment due next class on Canvas
1. With std_coverage.txt file, make a plot of your choice to show chaning abundance of bins across the samples. Provide the code you use if using R. 
