# CL2: High-Performance Computing

This session, you will get acquainted with working on the TXST High-Performance Computing (HPC) cluster: [LEAP2](https://doit.txst.edu/hpc.html).

---
## 🧠 Learning Objectives

By the end of this computer lab, you should be able to:

- Access remote HPC platforms
- Get familiar with file transfers from and to HPC clusters

---

An HPC cluster, as the name suggests, is a group of interconnected computers that work together as a single system, providing the computational resources needed to process large datasets or resolve complex computational problems. You can think of an HPC cluster as a supercomputer built from many individual computers working in concert.

Each computer within a cluster is called a *node*, and, as you will see below, different types of nodes serve different purposes. Because an HPC cluster consists of multiple nodes, large computational jobs can be divided into smaller tasks to be run in parallel, dramatically reducing the time required to complete them. 

An HPC cluster consists of several key components:
- **Login nodes:** Whenever you login into a cluster, you are taken to a login node, aka a *head* node. These nodes are not intended for running jobs. Instead, they are used to navigate your files, inspect and manage data, prepare and submit jobs, monitor their progress, and verify that they completed successfuly.
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
Type `passwd' and press ENTER to start. You should get a message like this one:

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

## Using SLURM
## Differentiating Login from Compute Nodes
## Transferring Data


