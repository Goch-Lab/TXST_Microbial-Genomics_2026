# CL2: High-Performance Computing

This session, you will get acquainted with working on the TXST High-Performance Computing (HPC) cluster: [LEAP2](https://doit.txst.edu/hpc.html).

---
## 🧠 Learning Objectives

By the end of this computer lab, you should be able to:

- Access remote HPC platforms
- Identify key components of HPC servers
- Learn good practices of working on HPC clusters
- Execute jobs through a job scheduler
- Get familiar with file transfers from and to HPC clusters

---

An HPC cluster, as the name suggests, is a group of interconnected computers that work together as a single system, providing the computational resources needed to process large datasets or resolve complex computational problems. You can think of an HPC cluster as a supercomputer built from many individual computers working in concert.

Each computer within a cluster is called a *node*, and, as you will see below, different types of nodes serve different purposes. Because an HPC cluster consists of multiple nodes, large computational jobs can be divided into smaller tasks to be run in parallel, dramatically reducing the time required to complete them. 

An HPC cluster consists of several key components:
- **Login nodes:** Whenever you login into a cluster, you are taken to a login node, aka a *head node*. These nodes are not intended for running jobs. Instead, they are used to navigate your files, inspect and manage data, prepare and submit jobs, monitor their progress, and verify that they completed successfuly.
- **Compute nodes:** These are the heavy lifters. Compute nodes are equipped with high-core-count CPUs and/or GPUs designed to handle intense workloads. This is where the jobs you submit are executed.
- **Interconnect network:** A dedicated, high-bandwidth (fast), low-latency (with little delay) network that connects all nodes, allowing them to communicate efficiently during parallel computations.
- **Shared storage:** A distributed storage system that enables all nodes to access the same files simultaneously, allowing them to read from and write to large datasets as jobs are running.

<img width="750" height="500" alt="image" src="https://lh7-us.googleusercontent.com/docsz/AD_4nXeCyNz5uNzvZb07vy0WpbRUzRaXaqnZqDUNWrZ2zJWQd3tICCOA9NzKKB_fiQWbAgBwdgWphBG28KoCD64arb3sxyR2yRdG3CM5t8qiTUSBB37w2h6vCKWwGJfU1PhgXAU-9Ca1GNmZiPITpEyR5N__Hznc?key=HnlHoIeLkXgfaK2MXC-KBw" />

## Accessing LEAP2
Connecting to an HPC cluster is typically done with a program known as “SSH” (<ins>S</ins>ecure <ins>SH</ins>ell), which is accessed through the Terminal (Linux and Mac) or a Terminal emulator (Windows; *see [CL1](./CL1.md)*).

SSH allows us to connect to a remote computer/cluster over a network to execute commands and transfer files from/to this remote platform. To access TXST's remote cluster LEAP2, SSH requires the *domain* (web address) of the cluster. You should have received an email from Shane Flaherty with your login information and the LEAP2 domain.

Open your Terminal to connect to LEAP2:

```bash
ssh <netID>@leap2.txstate.edu
```

If this is your first time connecting to LEAP2, you will probably see a similar message as the one below:

```bash
The authenticity of host 'leap2.txstate.edu' can't be established.
RSA key fingerprint is 2a:b6:f6:8d:9d:c2:f8:2b:8c:c5:03:06:a0:f8:59:12.
Are you sure you want to continue connecting (yes/no)?
```

This is your computer warning you that you are about to connect to another unrecognized computer, type `yes` and press ENTER to proceed. This will add LEAP2 to your "known hosts", and you should not see the message again in the future.

You should now be prompted to input your password. Your first password can be found in the email from Shane. Type it in carefully (or better yet, copy-paste it) because no characters will appear on the screen for you to see as you type. Then, press ENTER.

If you entered your password appropriately, congratulations! You are now connected to the HPC! 🎉

Now, if it is your first time accessing LEAP2, we will change your password as per Shane's request.
Type `passwd` and press ENTER to start. You should get a message like this one:

```bash
Changing password for user <netID>.
(current) LDAP Password: 
```

Type in the password provided by Shane Flaherty and press ENTER. You should see this:

```bash
New password: 
```

Type in your new password and press ENTER. You will see the following:

```bash
New password: 
```

Type in the new password again and press ENTER. If done successfully, you should see the following:

```bash
passwd: all authentication tokens updated successfully.
```

Log out of LEAP2:

```bash
exit
```

Notice that you are now out of the cluster and back in your local computer. Log in again using your new login information:

```bash
ssh <netID>@leap2.txstate.edu
```

Now, if you are not in campus, you will need to connect to TXST [VPN](https://services.txst.edu/TDClient/39/ITAC/Requests/Service/86/Virtual-Private-Network-VPN) (<ins>V</ins>irtual <ins>P</ins>rivate <ins>N</ins>etwork) first.

## Using SLURM
Now that you are becoming familiar with HPC systems, you might be wondering: how do I run things on here?

Remember that to run "stuff", you need to do it on the compute nodes. For users to access compute nodes to execute jobs, HPC clusters typically rely on *job scheduling systems*. A job scheduling system, or simply job scheduler, is a software that automatically runs, stops, and manages computer tasks at set times or when specific events happen. This way, all users can run their jobs in an organized manner and with a fair allocation of computational resources.

The job scheduler implemented on LEAP2 is SLURM (<ins>S</ins>imple <ins>L</ins>inux <ins>U</ins>tility for <ins>R</ins>esource <ins>M</ins>anagement).

<img width="575" height="431" alt="image" src="https://static.wikia.nocookie.net/enfuturama/images/8/80/Slurm-1-.jpg/revision/latest?cb=20060626052801" />

A job scheduler like SLURM is responsible for a few key tasks:

- Understanding what resources are available on the cluster. This includes the number of available compute nodes, the size of those compute nodes and the jobs currently running on them.
- Queuing and allocating jobs. Jobs are allocated to compute nodes to run based on the requested resources and the resources available.
- Monitoring and reporting the status of jobs. So that the user can know what jobs are in the queue, which jobs are running, which jobs have failed, which jobs completed successfully, etc.

To tell SLURM to run a job, you need to write a *job submission script* in which you specify the computational resources (number of CPUs, RAM, time) needed to complete the tasks successfully. These resources are specify at the top of your job script through some SLURM directives. These directives are indicated by lines starting with `#SBATCH`.

For example, the directive `#SBATCH --job-name=blast` will tell SLURM that you have named this job "blast", which can help make it easier for you to monitor your job and its outputs. Some `#SBATCH` directives also have a shorthand notation, e.g., `#SBATCH -J blast` is the same as the prior directive since -J and --job-name are interchangeable.

If you look at the SLURM documentation you will notice that there are A LOT of directives you can use. But fortunately, there are only a few key directives you need to get your job running. We will cover some of the more common SLURM directives below. You can also find some of the most useful directives in the [LEAP2 user guide](https://itrcstats2.itrc.txstate.edu/wiki/index.php/LEAP2_Cluster_User_Guide#SLURM_batch_directives).

The best way to understand how to use the SLURM directives to allocate the required resources for your job is to provide try some examples. Single-CPU (non-parallel) jobs are often simple commands that can be run on a single CPU:

```bash
vim single_cpu_job.sh 
```

Change to the insert mode and copy-paste the following script:

```bash
#!/bin/bash
#SBATCH --job-name=singlecpu
#SBATCH --partition=shared
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=1:00:00
#SBATCH --mem=5G

# Get started
echo "Job started on $(hostname) at $(date)"

#Variables

#Commands
sleep 30
echo "Hello World!"

# Finish up
echo "Job Ended at $(date)"
```

Escape from the insert mode, write and quit from vim as we learned before.

This script might look like a lot, but it really consists of four main sections. Let's break it down:

- Header. This section includes the first line and the `#SBATCH` directives. The firs line (`#!/bin/bash`) is specifying that this is a Bash script (hence *.sh). The `#SBATCH` directives are specifying the resources you are requesting for this job:
  - `--job-name=singlecpu`: assigns the name "singlecpu" to your job.
  - `--partition=shared`: determines the queue of the cluster your submitting your job too. "Shared" is the default partition. You might request for a different partition based on your computational needs. Read more [here](https://itrcstats2.itrc.txstate.edu/wiki/index.php/LEAP2_Cluster_User_Guide#Partitions).
  - `--nodes=1`: requests to run your job on a single compute node.
  - `--ntasks-per-node=1`: requests one CPU per requested node.
  - `--time=1:00:00`: sets the maximum running time to 1 h.
  - `--mem=5G`: sets the maximum memory (RAM) to 5 Gb.
- "Get started" section. This section prints (`echo`) a statement that records the node (`$(hostname)`) on which your job runs and the start date and time (`$(date)`).
- Variables section. Empty in this example, but that we will use later on.
- Commands section. This section is destined for the commands you want to run. In this case, the job will sleep for 30 s (`sleep 30`) before printing "Hello world!" (`echo "Hello World!"`).
- "Finish up" section. This section prints (`echo`) a statement that records the node (`$(hostname)`) on which your job ran and the finish date and time (`$(date)`). Along with the "get started" section, this section allows you to determine how long your job ran for.

To execute your job just submit your script to SLURM:

```bash
sbatch single_cpu_job.sh 
```

You will then be given a message with the ID for that job:

```bash
Submitted batch job XXXX
```

In this example, the job ID is represented by XXXX; keep record of it. You can also keep track of the submitted jobs:

```bash
squeue 
```

> [!Tip]
> You might have noticed a pattern: all SLURM commands start with S!

The last command will print the jobs submitted by all LEAP2 users. That is probably not what you want. To monitor your jobs, specify your user name:

```bash
squeue -u <netID>
```

You can also check the status of a specific job:

```bash
squeue --job <XXXX>
```

At this point, your job has probably completed, so you most likely will see an empty table.

By default, error messages and statements printed by your commands will be stored into a SLURM output file. Once the job has finished, check the content of your working directory:

```bash
ls
```

You should see a file that loos something like `slurm-XXXX.out`. Examine the content of the file using one or more of the commands you have learned before and make sure that it all makes sense.

It is also possible to cancel a running job:

```bash
scancel <XXXX>
```

## Differentiating Login from Compute Nodes
It is important to remember not to run jobs on the login nodes. How can we distinguish them from compute nodes then?

Pay attention to your LEAP2 session. The line where your cursor currently is should look something like this:

```bash
[netID@login1 ~]$
```

This is telling you that the user (`netID`) is currently at login node 1 (`login1`); you might be in a different login node. Keep in mind that different HPC platforms name their login and compute nodes differently. Now try this:

```bash
top
```

This command lets you explore all the jobs/tasks running on the login node you are currently at. Can you find yourself? If you can, great! That means other people on the same node can also find you. If you happen to run some computational-demanding task on the login node, you may slow down things for other uses on the same node. Even worse: they can track you down! Press `q` to quit.

If there are some jobs or tasks that demand some resources but only for a short time, and if you want to visually keep track of what's happening, you can run them interactively on the compute nodes. To do this, we ask SLURM to start an interactive shell:


```bash
sinteractive -p shared -n 2 --mem-per-cpu=5G --time=1:00:00
```

Looks familiar? This command is making use of the SLURM directives we learn about before, in this case we are using the 1-letter notations. `-p shared` is asking to start the shell in the partition "shared", `-n 2` is requesting to use 2 CPUs and 5 Gb of RAM in each of them (`--mem-per-cpu=5G`) for 10 Gb of memory in total, and `--time=1:00:00` is setting the maximum running time of the interactive session for 1 h.

Once you run the command, SLURM sends your request to the queue:

```bash
Waiting for JOBID XXXX to start
```

Once your interactive session starts, you will notice a change in your session:

```bash
[netID@compute-108 ~]$
```

Here you are able to run any tasks without bothering other users and without disruptions (as long as you stay within the requested computational resources).


## Transferring Data
It is possible to transfer files (e.g., data, scripts, outputs) from LEAP2 to your computer and the other way around. Let's explore how.

Start by creating a file:

```bash
vim some_file_name.txt
```

You are now familiar with vim, so feel free to add any text to it. Get the path to your current location (where you created the file):

```bash
pwd
```

Open another Terminal tab or window. In the new tab, without logging into LEAP2, run the following command:

```bash
scp <netID>@leap2.txstate.edu:/path/to/some_file_name.txt /path/to/desired/location
```

Make sure to replace some of the values above for the right ones. If done correctly, you will be asked for your LEAP2 password. Enter it and the download will start. Once completed, look for the dowloaded file, open it either from the Terminal using vim or with another text editor installed on your computer (e.g., TextEdit). Save it with a different file name.


Go back to the Terminal window, the one not signed on LEAP2. Upload the file to LEAP2:

```bash
scp /path/to/another_file_name.txt <netID>@leap2.txstate.edu:/path/to/desired/location
```

Go back to the LEAP2 window and go to the location where you uploaded the new file. Inspect its content to ensure the transfer was successful.
