# CL3: Sequence Data QC

This session, we will continue to get used to the command line while learning how to quality control (QC) raw read sequence data.

---
## 🧠 Learning Objectives

By the end of this computer lab, you should be able to:

- Understand the fastq format
- Organize files in a working directory
- Install bioinformatic tools using Conda
- Evaluate quality of raw sequence data

---
  
The first step in any genomics workflow is to prepare raw sequence data for our downstream analyses. In most cases, we will get our raw reads back from the sequencing facility in fastq-formatted files.

## What is a fastq file?

The fastq format has 4 lines per sequence: 
* The sequence identifier (header), preceded by an “@” character
* The nucleic acid sequence itself
* A “+” character sometimes followe by notes
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

## Setting Up Working Directory and Environment

First, we need to set up our working directory. Log in to LEAP2 as you learned in the previous lab. Create a directory called "microbial_genomics" and move to that directory:

```bash
mkdir microbial_genomics
cd microbial_genomics
```

Download the data we will be working with:

```bash
wget https://raw.githubusercontent.com/Goch-Lab/TXST_Microbial-Genomics_2026/main/data/02_sequenceQC/data_dir.tar.gz
```

If `wget` does not work, try `curl` instead:

```bash
curl -L -O https://raw.githubusercontent.com/Goch-Lab/TXST_Microbial-Genomics_2026/main/data/02_sequenceQC/data_dir.tar.gz
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

We now need to installthe programs we will be using with Conda:

```bash
conda create -y -n seqQC -c conda-forge -c bioconda -c defaults cutadapt fastqc trimmomatic multiqc
```

This will also take a while. It will create an environment named "seqQC" containing the programs utadapt, FastQC, Trimmomatic, and MultiQC. These programs will be installed from the conda-forge, bioconda, and default channels.

## Checking Read Sequence Data Quality

We can now activate our Conda environment:

```bash
conda activate seqQC
```

You now should be able to run the programs you installed in that environment. Have a look at the help of FastQC:

```bash
fastqc -h
```

Quit by pressing "q". Move back to your working directory, and create a directory for the first step:

```bash
cd microbial_genomics
mkdir fastqc
cd fastqc
```

To run FastQC, we need to create a SLURM script:

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

Take your time to go through the script and to understand what every line is doing. You might notice a few differences compared to the last SLURM script we ran. For example, `source ~/.bashrc` is making sure that your environment configuration in the login nodes is carried over to the computing nodes, including your Conda environments. The variables section now defines a variable, `$READS_DIR`, that is being called by the command below. Go ahead and submit the script:

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
scp '<netID>@leap2.txstate.edu:/home/<netID>/microbial_genomics/fastqc/*.html' </path/to/desired/location>
```

Open one of the files and try to make sense of it. It should be a file similar to [this one](https://htmlpreview.github.io/?https://github.com/Goch-Lab/TXST_Microbial-Genomics_2026/blob/main/data/02_sequenceQC/B1_sub_R1_fastqc.html). The instructor will walk you through it at some point.

Instead of checking each file individually, we can use the tool `multiqc` to produce a report of all the combined outputs. Go back to your LEAP2 window and let's run MultiQC interactively:

```bash
cd </path/to>/fastqc
sinteractive -p shared -n 1 --mem-per-cpu=5G --time=1:00:00
conda activate seqQC
multiqc .
conda deactivate
exit
```

Again, we will need to download the HTML file from another terminal tab/window to the local computer:

```bash
scp <netID>@leap2.txstate.edu:/home/<netID>/microbial_genomics/fastqc/multiqc_report.html </path/to/desired/location>
```

The file should look like [this one](https://htmlpreview.github.io/?https://github.com/Goch-Lab/TXST_Microbial-Genomics_2026/blob/main/data/02_sequenceQC/multiqc_report.html). Explore it with detail and make sure you understand what you see. The instructor will explain at some point.

This concludes CL3!
