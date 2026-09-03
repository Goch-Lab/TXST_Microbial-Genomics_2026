# CL4: Sequence Data Processing

This session, we will continue to use the command line to process raw read sequence data for subsequent analyses.

---
## 🧠 Learning Objectives

By the end of this computer lab, you should be able to:

- Run multiple jobs simultaneously on an HPC cluster
- Explain how filtering/trimming decisions affect data retention
- Learn the importance of QC sequence data for downstream analyses

---

## Trimming Adapters with Cutadapt 

Let's go back into our `microbial_genomics` directory and create a new directory called `cutadapt`:

```bash
cd ..
mkdir cutadapt
cd cutadapt/
```

We will now run Cutadapt on paired-end mode because, as you might remember from the lecture, in most cases reads are sequenced in both forward and reverse directions, meaning each forward read should have a paired read. We will use the following SLURM script to process all read pairs at the same time:

```bash
vim cutadapt.sh
```

Copy-paste the following:

```bash
#!/bin/bash
#SBATCH --partition=shared
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=1:00:00
#SBATCH --mem=20G
#SBATCH --array=0-19

# Get started
echo "Job started on $(hostname) at $(date)"

source ~/.bashrc
conda activate seqQC

#Variables
prefixes()
for f in ../data/*_R1.fq; do
        p=$(echo $f | cut -f3 -d'/' | cut -f1 -d'_')
        prefixes+=("$p")
done

export R1=../data/${prefixes[$SLURM_ARRAY_TASK_ID]}_sub_R1.fq
export R2=../data/${prefixes[$SLURM_ARRAY_TASK_ID]}_sub_R2.fq
export R1_AD=^GTGCCAGCMGCCGCGGTAA...ATTAGAWACCCBDGTAGTCC
export R2_AD=^GGACTACHVGGGTWTCTAAT...TTACCGCGGCKGCTGGCAC
export MINL=215
export MAXL=285
export OUT1=${prefixes[$SLURM_ARRAY_TASK_ID]}_sub_R1_trimmed.fq
export OUT2=${prefixes[$SLURM_ARRAY_TASK_ID]}_sub_R2_trimmed.fq

#Commands
cutadapt -a $R1_AD -A $R2_AD -m $MINL -M $MAXL --discard-untrimmed -o $OUT1 -p $OUT2 $R1 $R2

# Finish up
conda deactivate

echo "Job Ended at $(date)"
```

This script is running a job array. A job array is just a set of jobs for which the same task is run for different variables (e.g., a file). Every job in the array is assigned an index, based on the number of jobs running in the array. In this case, the array has 20 jobs (0-19), one for each pair of read files. Each value from 0 to 19 is assigned for each of the jobs being run. The index can be accessed through a SLURM variable (`$SLURM_ARRAY_TASK_ID`). To assign each pair of reads, we use Bash to first define an empty list `prefixes`. We then use a `for` loop to go through each of the R1 files. In that loop, we use a command to store a prefix from each R1 file (`p=$(echo $f | cut -f3 -d'/' | cut -f1 -d'_')`) in the variable `$p`. We then add the variable `$p` to our prefixes list in each iteration.

The Cutadapt command specifies the primers for the forward reads with the `-a` flag, giving it the forward primer (in normal orientation), followed by three dots (required by Cutadapt to know they are “linked”, with bases in between them, rather than right next to each other), then the reverse complement of the reverse primer. For the reverse reads, specified with the `-A` flag, we give it the reverse primer (in normal 5’-3’ orientation), three dots, and then the reverse complement of the forward primer. Both of those have a ^ symbol in front at the 5’ end, indicating they should be found at the start of the reads (which is the case with this particular setup). 

The minimum read length (set with `-m`) and max (set with `-M`) were based roughly on 10% smaller and bigger than would be expected after trimming the primers. The flag `--discard-untrimmed` states to throw away reads that don’t have these primers in the expected locations. Finally, `-o` specifies the output of the forward reads, `-p` specifies the output of the reverse reads, and the input forward and reverse are provided as positional arguments in that order.

>[!NOTE]
> These types of settings will be different for data generated with different sequencing, i.e., not 2x300, and different primers sets. 

Run the script:

```bash
sbatch cutadapt.sh
```

Keep track of the job until it finishes:

```bash
squeue -u <netID>
```

Once it finishes, you should see all trimmed files and the SLURM output files. Explore one of the SLURM files to get an idea of how things went.:

```bash
less slurm-XXXXXXX_X.out
```

Press "q" to quit.

## Assess Data Retention and Changes in Trimmed Files 

Use `grep` to look at what fraction of reads were retained in each sample:

```bash
grep "passing" *.out
```

Now look at what fraction of bp were retained in each sample:

```bash
grep "filtered" *.out
```

Would you say we lost little or a lot of data?

Let's take a quick look to see that the primers were trimmed off:

```bash
### R1 BEFORE TRIMMING PRIMERS
head -n 2 ../data/B1_sub_R1.fq
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
head -n 2 ../data/B1_sub_R2.fq
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

Let's use FastQC and MultiQC again to see the improvement in the outputs:

```bash
cd ../fastqc/
mkdir trimmed
cd trimmed/
cp ../fastqc.sh .
```

You should now work independently to edit the script so you can run it with the trimmed reads. Submit it, make sure it ran successfully to completion and run MultiQC in an interactive shell. Once finished, you may want to rename the MultiQC, download it to the local computer and open it in a web browser for comparison with the report from the raw data.

This is the end of CL3!
