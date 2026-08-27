# CL3: Sequence Data QC

This session, we will continue to get used to the command line while learning how to quality control (QC) raw read sequence data and process them for subsequent analyses. 

---
## 🧠 Learning Objectives

By the end of this computer lab, you should be able to:

- Understand the fastq format
- Evaluate raw sequence data quality
- Explain how filtering/trimming decisions affect data retention
- Learn the importance of QC sequence data for downstream analyses

---
  
The first step in any genomics workflow is to prepare raw sequence data for our downstream analyses. In most cases, we will get our raw reads back from the sequencing facility in fastq-formatted files.

## What is a fastq file?

The fastq format has 4 lines per sequence: 
* The sequence identifier (header), preceded by an “@” character
* The nucleic acid sequence itself
* A “+” character and possibly the header information repeated or other notes
* The quality score for the basecalling of each individual nucleotide, which must be the same number of characters as nucleotides in the sequence

![Fastq file format](https://github.com/user-attachments/assets/e7a64482-ccae-4ee4-99a8-4bb83141b448)

Quality score is a measure of how confident the software was when it called that particular base position during sequencing. Different sequencing technologies employ different sets of [ASCII characters](https://www.ascii-code.com/) to represent their quality scores, with each character representing a quality score. Current Illumina sequencing technologies, for example, use the offset starting at character number 33 (Phred+33), which is "!".

<img width="3436" height="656" alt="image" src="https://training.galaxyproject.org/training-material/topics/sequence-analysis/faqs/images/fastq-quality-encoding.png" />

It is important to know that this is not a perfect system, as there are still confounding factors like polymerase error and other systematic errors that will not show up in the quality score information, but performing some quality-based filtering after sequencing is essential.

Demultiplexing refers to the step in processing where we use barcode information to know what sequences came from which samples after being sequenced together. Barcodes are unique sequences attached to each sample's DNA fragments before the samples got all pooled together. Demultiplexing is typically done by sequencing facilities nowadays. It is important for you to know that this step takes place before QC. We will *not* go over demultiplexing. 

---

## The Sequence Data

Mike and his team were exploring an underwater mountain ~3 km down at the bottom of the Pacific Ocean that serves as a low-temperature (~5-10°C) hydrothermal venting site. This amplicon dataset was generated from DNA extracted from crushed basalts collected from across the mountain with the goal of characterize the microbial communities of these deep-sea rocks. No one had ever been there before, so as is often the purpose of marker-gene sequencing, this was just a broad-level community survey. The sequencing was done on an Illumina MiSeq platform with 2 x 300 bp paired-end sequencing, using primers targeting the V4 region (~291 bp) of the 16S rRNA gene. There are 20 samples total: 4 extraction “blanks” (nothing added to DNA extraction kit), 2 bottom-water samples, 13 rocks, and one biofilm scraped off a rock. 

In the following figure, overlain on the map are the rock sample collection locations, and the panes on the right show examples of the 3 distinct types of rocks collected: 1) basalts with highly altered, thick outer rinds (>1 cm); 2) basalts that were smooth, glassy, thin exteriors (~1-2 mm); and 3) one calcified carbonate.

<img width="800" height="436" alt="image" src="https://github.com/user-attachments/assets/aa197211-0392-4b63-998a-6e32de69efb5" />

This work was published and you can read more about it [here](https://www.frontiersin.org/journals/microbiology/articles/10.3389/fmicb.2015.01470/full).

## Checking Read Sequence Data Quality
First, we need to set up our working directory. Log in to LEAP2 as you learned in the previous lab. Create a directory called "microbial_genomics" and move to that directory:

```bash
mkdir microbial_genomics
cd microbial_genomics
```

Download the data we will be working with:

```bash
wget https://raw.githubusercontent.com/morgansobol/MicrobialGenomics-TXST-2025/main/data/02_sequenceQC/data_dir.tar.gz
```

If `wget` does not work, try curl instead:

```bash
curl -L -O https://raw.githubusercontent.com/morgansobol/MicrobialGenomics-TXST-2025/main/data/02_sequenceQC/data_dir.tar.gz
```

Now this is a compressed file, also called a "tarball". To unpack it, like opening a zip file, we run:

```bash
tar -xzvf data_dir.tar.gz
```

This command combines multiple flags: `x` stands for "extract", `z` for "gzipped", `v` for "verbose" (print details of what is happening to the prompt), and `f` for "file". This should have unpacked a new directory called `data_dir`. Change the directory name to "data":

```bash
mv data_dir data
```

Remove the tarball to free space:

```bash
rm data_dir.tar.gz
```

Let's set up the rest of our environment to process the data. To install the required programs, we will use Conda. Conda is an open-source tool that manages software packages and virtual environments for multiple programming languages. We will download and install a comprehensive distribution of Conda called Anaconda. This should make easier the installation of programs throughout the course:

```bash
curl -O https://repo.anaconda.com/archive/Anaconda3-2026.07-1-Linux-x86_64.sh
bash Anaconda3-2026.07-1-Linux-x86_64.sh
```

This will start a prompt that should look something like this:

```bash
Welcome to Anaconda3 2026.07-1

In order to continue the installation process, please review the license
agreement.
Please, press ENTER to continue
>>> 
```

Go ahead and press enter. It will then show something like this: 

```bash
By continuing installation, you hereby consent to the Anaconda Terms of Service available at https://anaconda.com/legal.


Do you accept the license terms? [yes|no]
>>>
```

Type "yes" and press enter. It will show something like the following:

```bash
Anaconda3 will now be installed into this location:
/home/netID/anaconda3

  - Press ENTER to confirm the location
  - Press CTRL-C to abort the installation
  - Or specify a different location below

[/home/netID/anaconda3] >>> 
```

Press enter. It will then run for a while printing a bunch of messages to the prompt. These are all packages Anaconda is installing in your environment.  It will then print the following:

```bash
Downloading and Extracting Packages:

Preparing transaction: done
Executing transaction: done
installation finished.
Do you wish to update your shell profile to automatically initialize conda?
This will activate conda on startup and change the command prompt when activated.
If you'd prefer that conda's base environment not be activated on startup,
   run the following command when conda is activated:

conda config --set auto_activate_base false

Note: You can undo this later by running `conda init --reverse $SHELL`

Proceed with initialization? [yes|no]
[no] >>>
```
Type "yes" and press enter. This will add Conda to your default environment.
Log out and back in onto LEAP2:

```bash
exit
ssh <netID>@leap2.txstate.edu
```

We now need to install the programs the programs we will be using using Conda:

```bash
conda create -y -n seqQC -c conda-forge -c bioconda -c defaults cutadapt fastqc trimmomatic multiqc
```

This will also take a while. It will create an environment named "seqQC" containing the programs cutadapt, fastqc, trimmomatic, and multiqc. These programs will be installed from the conda-forge, bioconda, and default channels. We can now activate the environment:

```bash
conda activate seqQC
```

You now should be able to run the programs you installed in that environment. Have a look at the help of fastqc:

```bash
fastqc -h
```

Quit by pressing "q". Move back to your working directory, and create a directory for the first step:

```bash
cd microbial_genomics
mkdir fastqc
cd fastqc
```

To run fastqc, we need to create a SLURM script:

```bash
vim fastqc.sh
```

Copy-paste the following into the script:

```bash
#!/bin/bash
#SBATCH --job-name=fastqc
#SBATCH --partition=shared
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=1:00:00
#SBATCH --mem=20G

# Get started
echo "Job started on $(hostname) at $(date)"

source ~/.bashrc
conda activate seqQC

#Variables
export READS_DIR=../data

#Commands
fastqc ${READS_DIR}/*.fq -o .

# Finish up
conda deactivate

echo "Job Ended at $(date)"
```

Take your time to go through the script and to understand what every line is doing. You might notice a few differences compared to the last SLURM script we ran. For example, `source ~/.bashrc` is making sure that your environment configuration in the login nodes is carried over to the computing nodes, including your Conda environments. The variables section now defines a variable, `$READS_DIR`, that is being called by the command. Go ahead and submit the script:

```bash
sbatch fastqc.sh
```

Keep track of the job until it finishes:

```bash
squeue -u <netID>
```

Verify there were no errors by exploring the content of the SLURM output file:

```bash
less slurm-XXXXXXX.out
```

Quit by pressing "q". Explore the content of your working directory:

```bash
ls
```

You should see a bunch of files if the job completed successfully. The outputs are HTML files (*.html) that you can view on a web browser. However, you will need to download the HTML files from another terminal tab/window (not on LEAP2) to the local computer to do so:

```bash
scp '<netID>@leap2.txstate.edu:/home/<netID>/microbial_genomics/fastqc/*.html' /path/to/desired/location
```

Open one of the files and try to make sense of it. It should be a file similar to [this one](../data/02_sequenceQC/B1_sub_R1_fastqc.html).


However, instead of checking each file individually, we can instead use the tool `multiqc` which will aggregate all of our results together.
```bash
multiqc .
open multiqc_report.html
```
Or like so if on mobaXterm
```bash
explorer.exe multiqc_report.html
```

Now we need to trim the primers off the reads and low-quality bases, both from read 1 and read 2, to improve their quality.


## 🧪 Exercise 2: Removing primers with Cutadapt 

Let's go back one directory, into the working_dir, and create a new directory called cutadapt.
```bash
cd ..
mkdir cutadapt
cd cutadapt/
```

Now we will run cutadapt on paired-end mode, because, remember from lecture, most cases reads are sequenced in the forward and reverse direction, meaning each forward read should have a paired read. We will try with one sample first before running as a loop to process all samples at once. 

```bash
cutadapt -a ^GTGCCAGCMGCCGCGGTAA...ATTAGAWACCCBDGTAGTCC \
         -A ^GGACTACHVGGGTWTCTAAT...TTACCGCGGCKGCTGGCAC \
         -m 215 -M 285 --discard-untrimmed \
         -o B1_sub_R1_trimmed.fq -p B1_sub_R2_trimmed.fq \
         ../../data_dir/B1_sub_R1.fq ../../data_dir/B1_sub_R2.fq 
```

We are specifying the primers for the forward read with the -a flag, giving it the forward primer (in normal orientation), followed by three dots (required by cutadapt to know they are “linked”, with bases in between them, rather than right next to each other), then the reverse complement of the reverse primer. 

Then for the reverse reads, specified with the -A flag, we give it the reverse primer (in normal 5’-3’ orientation), three dots, and then the reverse complement of the forward primer. Both of those have a ^ symbol in front at the 5’ end, indicating they should be found at the start of the reads (which is the case with this particular setup). 

The minimum read length (set with -m) and max (set with -M) were based roughly on 10% smaller and bigger than would be expected after trimming the primers. 

--discard-untrimmed states to throw away reads that don’t have these primers in them in the expected locations. 

Then -o specifies the output of the forward reads, -p specifies the output of the reverse reads, and the input forward and reverse are provided as positional arguments in that order.

>[!NOTE]
> These types of settings will be different for data generated with different sequencing, i.e. not 2x300, and different primers sets. 

Ok, let's take a quick look to see that the primers were trimmed off. 
```bash
### R1 BEFORE TRIMMING PRIMERS
head -n 2 ../../data_dir/B1_sub_R1.fq
# @M02542:42:000000000-ABVHU:1:1101:8823:2303 1:N:0:3
# GTGCCAGCAGCCGCGGTAATACGTAGGGTGCGAGCGTTAATCGGAATTACTGGGCGTAAAGCGTGCGCAGGCGGTCTTGT
# AAGACAGAGGTGAAATCCCTGGGCTCAACCTAGGAATGGCCTTTGTGACTGCAAGGCTGGAGTGCGGCAGAGGGGGATGG
# AATTCCGCGTGTAGCAGTGAAATGCGTAGATATGCGGAGGAACACCGATGGCGAAGGCAGTCCCCTGGGCCTGCACTGAC
# GCTCATGCACGAAAGCGTGGGGAGCAAACAGGATTAGATACCCGGGTAGTCC

### R1 AFTER TRIMMING PRIMERS
head -n 2 B1_sub_R1_trimmed.fq
# @M02542:42:000000000-ABVHU:1:1101:8823:2303 1:N:0:3
# TACGTAGGGTGCGAGCGTTAATCGGAATTACTGGGCGTAAAGCGTGCGCAGGCGGTCTTGTAAGACAGAGGTGAAATCCC
# TGGGCTCAACCTAGGAATGGCCTTTGTGACTGCAAGGCTGGAGTGCGGCAGAGGGGGATGGAATTCCGCGTGTAGCAGTG
# AAATGCGTAGATATGCGGAGGAACACCGATGGCGAAGGCAGTCCCCTGGGCCTGCACTGACGCTCATGCACGAAAGCGTG
# GGGAGCAAACAGG


### R2 BEFORE TRIMMING PRIMERS
head -n 2 ../../data_dir/B1_sub_R2.fq
# @M02542:42:000000000-ABVHU:1:1101:8823:2303 2:N:0:3
# GGACTACCCGGGTATCTAATCCTGTTTGCTCCCCACGCTTTCGTGCATGAGCGTCAGTGCAGGCCCAGGGGACTGCCTTC
# GCCATCGGTGTTCCTCCGCATATCTACGCATTTCACTGCTACACGCGGAATTCCATCCCCCTCTGCCGCACTCCAGCCTT
# GCAGTCACAAAGGCCATTCCTAGGTTGAGCCCAGGGATTTCACCTCTGTCTTACAAGACCGCCTGCGCACGCTTTACGCC
# CAGTAATTCCGATTAACGCTCGCACCCTACGTATTACCGCGGCTGCTGGCACTCACACTC

### R2 AFTER TRIMMING PRIMERS
head -n 2 B1_sub_R2_trimmed.fq
# @M02542:42:000000000-ABVHU:1:1101:8823:2303 2:N:0:3
# CCTGTTTGCTCCCCACGCTTTCGTGCATGAGCGTCAGTGCAGGCCCAGGGGACTGCCTTCGCCATCGGTGTTCCTCCGCA
# TATCTACGCATTTCACTGCTACACGCGGAATTCCATCCCCCTCTGCCGCACTCCAGCCTTGCAGTCACAAAGGCCATTCC
# TAGGTTGAGCCCAGGGATTTCACCTCTGTCTTACAAGACCGCCTGCGCACGCTTTACGCCCAGTAATTCCGATTAACGCT
# CGCACCCTACGTA
```

We are going to trim those again in the loop, so let's delete these trimmed files so as not to have duplicates.
```bash
rm *.fq
ls
```

Now, on to doing them all with a loop, here is how we can run it on all our samples at once. Since we have a lot of samples here, I’m redirecting the “stdout” (what’s printing the stats for each sample) to a file called *cutadapt_primer_trimming_stats.txt* so we can more easily view and keep track of if we’re losing a ton of sequences or not by having that information stored somewhere – instead of just plastered to the terminal window. We’re also going to take advantage of another convenience of cutadapt – by adding the extension .gz to the output file names, it will compress the files for us.

```bash
nano cutadapt.sh
```

Add this to the bash file you just created.
```bash
#!/usr/bin/env bash

DIR="../../data_dir/"

for R1 in ${DIR}/*_R1.fq; do
    # derive sample name by stripping path and suffix
    sample=$(basename "$R1" _R1.fq)
    R2="${DIR}/${sample}_R2.fq"

    echo "On sample: $sample"

    cutadapt \
        -a ^GTGCCAGCMGCCGCGGTAA...ATTAGAWACCCBDGTAGTCC \
        -A ^GGACTACHVGGGTWTCTAAT...TTACCGCGGCKGCTGGCAC \
        -m 215 -M 285 --discard-untrimmed \
        -o ${sample}_R1_trimmed.fq.gz \
        -p ${sample}_R2_trimmed.fq.gz \
        "$R1" "$R2" \
        >> cutadapt_primer_trimming_stats.txt 2>&1
done
```

- a = forward adapter
- A = reverse adapter
- m 215 = will discard all reads below 215 bp 
- M 285 = will discard all reads larger than 285 bp
- discard-untrimmed = discards reads in which no adapter match was found.
- o = output R1 
- p = output R2

Now run it
```bash
bash cutadapt.sh
ls
```

You should see all trimmed files here and the output stats file.
I typically like to have a file with all the sample names to use for various things throughout, so here’s making that file based on how these sample names are formatted. 
```bash
ls *_R1_trimmed.fq.gz | cut -f1 -d "_" > samples.txt
```

Now you can look through the output of the cutadapt stats file we made (“cutadapt_primer_trimming_stats.txt”) to get an idea of how things went. Here’s a little one-liner to look at what fraction of reads were retained in each sample (column 2) and what fraction of bps were retained in each sample (column 3):
```bash
paste samples.txt <(grep "passing" cutadapt_primer_trimming_stats.txt | cut -f3 -d "(" | tr -d ")") <(grep "filtered" cutadapt_primer_trimming_stats.txt | cut -f3 -d "(" | tr -d ")")
```

Great, so in all cases >90% of reads were kept. 

Let's also check again with FastQC/MultiQC to see how that improved the output
```bash
cd ../fastqc/
mkdir trimmed
cd trimmed/
fastqc ../../cutadapt/*.fq.gz -o .
multiqc .
open multiqc_report.html
```
With primers removed, we’re now ready to switch to R Studio and start using DADA2!

But, before we do that, let's prepare a new directory to work in with everything we need.
```bash
cd ../../
mkdir dada2
cd dada2
cp ../cutadapt/samples.txt .
ln -s ../cutadapt/*.fq.gz .
ls
```
I am introducing a new command here `ln`. This command creates a symbolic (hence -s) link, also known as a symlink or soft link. This is a special type of file that points to another file or directory. Symbolic links are commonly used to create shortcuts or aliases for files or directories located in the file system. This allows us to use those files in this directory, without having to make a duplicate, hard copy. 

Ok, now we can switch to R and process these reads (: 

```bash
pwd
```
We will need this path for R. 
