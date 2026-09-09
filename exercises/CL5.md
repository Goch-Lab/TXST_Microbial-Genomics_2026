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
conda create -n assembly -c bioconda -c conda-forge spades
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

1. Because we are working with the genome of a bacterial isolate genome (and not a metagenome), the read coverage is likely pretty high (>50x). We need to run our assembly in the `--isolate` mode.
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

We will use [Quast](https://quast.sourceforge.net/index.html) to asses quality of the resulting genome assembly. Create a Conda environment for Quast:

```bash
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
conda activate quast
quast.py contigs.fasta
conda deactivate
exit
```

Once finished, download the HTML report (`quast_results\results_<...>\report.html`) to the local computer. It should be similar to [this one](https://htmlpreview.github.io/?https://github.com/Goch-Lab/TXST_Microbial-Genomics_2026/blob/main/data/04_assembly/report.html).


## Interpreting Assembly Quality

Open the Quast report in an internet browser and inspect it. Notice the different metrics it reports, one of them is the N50. The N50 length is a proxy of contiguity. This is how it is calculated:

<img width="699" height="141" alt="N50 diagram" src="https://i0.wp.com/www.molecularecologist.com/wp-content/uploads/2017/03/Figure1b.jpg" />

Based on the Quast report, and what you have learned in the lectures, try to answer the following questions:

- What is the total assembly length? Is this equal to the genome size?
- How good is the N50?
- Is the assembly fragmented or contiguous?
- What metric(s) provide that kind of information?
- Would you consider this assembly to be of high quality?

Feel free to discuss with your classmates.

This concludes CL5!
