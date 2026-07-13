# CL1: Introduction to Command Line

This session, you will get comfortable using the Unix command line, which is an essential skill for working with large datasets, including genomic and other omic data.

---
## 🧠 Learning Objectives

By the end of this computer lab, you should be able to:

- Navigate the file system using `cd`, `ls` and `pwd`
- Create, move, copy, name and delete files and directories
- Use redirectors and wildcards
- Explore files using `head`, `tail`, `less`, `nano` and `cat`
- Edit text files directly from the shell

---

Let's establish some basics first. 

Linux and Mac users will find a Terminal program already installed on their computers. If you are using Windows, I recommend downloading [MobaXTerm](https://mobaxterm.mobatek.net/), but other software are available. We can also get you set up on your personal computer later if you own a Window/PC. 

The *Terminal* is a text input and output environment where we can execute commands and see the output. In other words, it is the "window" in which you enter the actual commands, and those commands are then interpreted and run by a *Shell*. 

The *Shell* is the program inside the terminal that actually processes the commands and returns the output. In most Linux and Mac operating systems, it uses a *Bash* shell, which is essentially its own programming language and what we will be using during this course. 

Different shells provide unique features and syntax: on macOS, the default is Zsh (formerly Bash) with Unix-style commands, while Windows primarily uses PowerShell and Command Prompt, with options to install Unix-like shells such as WSL or Git Bash.

In summary, think of it this way: terminal is the TV and shell is the program running the TV. 

Computers organize file locations in a hierarchical structure. Even though we might not notice, we are typically used to navigating through this stucture by clicking on various folders (also known as directories) in a Mac Finder or Windows Explorer window. Just like we need to select the right files in the appropriate locations in a Graphical User-Interface (GUI), we need to do the same when working at a command-line interface. What this means in practice is that each file and directory has its own “address”, and that address is called “path”.

Additionally, there are two special locations in all file systems: 🫜"root” and the user’s 🏠“home”. *Root* is where the file system of a computer starts, whereas *home* is where a user’s file system starts (this is where you should be now 📍). An *absolute* path is the one that indicates location based on root; whereas a *relative* path is based on the user's home.

<img width="1511" height="1799" alt="image" src="https://github.com/user-attachments/assets/d051f1d5-91ee-44b7-b175-9471422c174b" />

## 🧭 Navigating the File System

Before we begin exploring the command, let's get the tutorial files:

```bash
git clone https://github.com/Goch-Lab/TXST_Microbial-Genomics_2026.git
```

Congratulations! 🎉 
You just ran your first command, `git clone`, which specifically allows us to clone files and data from a GitHub repository. 

Before we look at other common commands, note a few keyboard commands that are very helpful:

- `Up Arrow`: shows your last command
- `Down Arrow`: shows your next command
- `Tab`: auto-completes your command
- `Ctrl + L`: clears the screen
- `Ctrl + C`: cancels a command
- `Ctrl + R`: stats a search for a command
- `Ctrl + D`: exits the terminal
- `history`: shows your command history

Let's explore other basics!

Check your current location by <ins>p</ins>rinting your <ins>w</ins>orking <ins>d</ins>irectory (i.e., where you currently are in the system).

```bash
pwd
```

It should look something like this `/home/[name]` and tell you that you are in the "home" location. 

Now <ins>l</ins>i<ins>s</ins>t the contents of your current directory using the `ls` command:

```bash
ls
```

You should see the directory we just cloned from GitHub called "TXST_Microbial-Genomics_2026"; we will explore this in a minute. 

Often, there are *hidden* files, typically configuration files, which begin with a dot (`.`). E.g., your Bash profile is configured by the file ~/.bash_profile. Configuration files do things like store settings and preferences for programs, determine what programs are "turned on" when you log in, and customize how your shell behaves. 

To see these *hidden* files use the `ls` command with the argument `-a`, to see <ins>a</ins>ll files, including hidden ones:

```bash
ls -a
```

You should see file names like these `.  ..  .bash_history  .bash_logout  .bash_profile  .bashrc  .emacs  example.txt  .kshrc  MicrobialGenomics-TXST-2025  .mozilla  .ssh`. 

You can also use `ls` to see the size (in bytes) of the files in your directories with the argument `-l`.

```bash
ls -l
```

I prefer to add `-lh`, the "h" prints the sizes in a <ins>h</ins>uman-readable format:

```bash
ls -lh
```

> [!TIP]
> On Linux and Mac, the `man` command is used to show the **manual** of any command that you can run in the terminal. So, if you wanted to know more about the `ls` command, you could run:

```bash
  man ls
```

If the `man` command is not included, you can just type the command that you want to know more about and then `--help` and you will get similar info:

```bash
  ls --help
```

You should be able to use the arrow keys, page up and down, or the space bar to scroll through. When you are ready to exit, just press `q`.

Let's move into our cloned GitHub directory, or <ins>c</ins>hange <ins>d</ins>irectories using the `cd` command. Move to the TXST_Microbial-Genomics_2026 directory and use `pwd` and `ls` to see where you are and what is there. **Type each line one at a time and press enter:**

```bash
cd TXST_Microbial-Genomics_2026/
pwd
ls
```

To move back one directory, simply use `..`:

```bash
cd ..
pwd
ls
```

Again, the `pwd` command should return `/home/[name]` and `ls` should return `.  ..  .bash_history  .bash_logout  .bash_profile  .bashrc  .emacs  example.txt  .kshrc  TXST_Microbial-Genomics_2026  .mozilla  .ssh`. 

You can also use `cd` to "jump" to other directories quickly:

```bash
cd TXST_Microbial-Genomics_2026/data/01_intro/
pwd
```

If you type `cd` alone, it will bring you all the way back to home from whichever location you are at.

```bash
cd
pwd
```

Let's go back to where we were before and try something else:

```bash
cd TXST_Microbial-Genomics_2026/data/01_intro/
pwd
```

When combined with `..`, you can move back multiple directories as well:

```bash
cd ../../
pwd
```

Where are you now? (Hint: you should be at `/home/[name]/TXST_Microbial-Genomics_2026`)

> [!TIP]
> When trying to specify a file or path, we can begin typing its name and then press the <ins>tab</ins> key to autocomplete it (try it out below). If there is only one possible way to finish what we started typing, it will complete it entirely for us. If there is more than one possible way to finish what we started typing, it will complete as far as it can, and then hitting tab twice quickly will show all the possible options. If tab-complete does not do either of those things, then we are either confused about where we are or what is where, or we may have misspelled the file name.

Let's try this out. Go back to your home with `cd`. Now, let's go back to the `01_intro/` directory, but this time we will use tab to fill out our path, not type it out:

```bash
cd MicrobialGenomics-TXST-2025/data/01_intro/
pwd
```

That was a brief intro to moving around the file system using the command line. Practice makes perfect, so practice this whenever you can, and it will eventually become natural. 

Summary of commands used:

| Command        | Description                                                       |
|----------------|-------------------------------------------------------------------|
| `pwd`          | Prints the path to the working directory                          |
| `ls`           | List directory contents                                           |
| `ls -a`        | List contents including hidden files (files beginning with a dot) |
| `ls -l`        | List contents with more info including permissions (long listing) |
| `ls -lh`       | List file sizes in human-readable format                          |
| `cd`           | Go to home                                                        |
| `cd ~`         | Go to home                                                        |
| `cd [dirname]` | Go to specified directory                                         |
| `cd ..`        | Change to parent directory                                        |

## 📁 Creating, Copying, Moving, Renaming and Removing Files & Directories
> [!WARNING]
> Using commands that create, copy or move files will <ins>**overwrite**</ins> them if there are other files with the same name. More importantly, using commands to delete files/directories will do so <ins>**permanently**</ins>. Use caution when using these commands!

We should still be in the `01_intro/` directory. Let's make a new directory inside this one. 
Start with <ins>m</ins>a<ins>k</ins>ing a new <ins>dir</ins>ectory using the `mkdir` command, then move into it:

```bash
mkdir practice
ls
cd practice
```

To create files in this directory, we will use the command `touch`. Then, use `ls` to check that the files were created:

```bash
touch sample1.txt
touch sample2.txt
touch sample3.txt
ls
```

Let's <ins>c</ins>o<ins>p</ins>y the second file and rename it. The first file called is the file you want to copy, the second is what you want to name the copy; you can think of it in a "from-to" manner:

```bash
cp sample2.txt sample2_copy.txt
ls
```

To make a copy and put it somewhere else, like in new subdirectory, we can change the second positional argument using a relative path. Make a new directory called "copies" then copy "Sample 3" there:

```bash
mkdir copies
cp sample3.txt copies/sample3_copy.txt
cd copies/
ls
```

Now we want to move the sample2_copy.txt file into the `copies/` directory. To do that, we will use the <ins>m</ins>o<ins>v</ins>e command `mv`), but remember the file is in the previous directory (hint: `..`). You can use a single `.` to specify that you want `mv` to move the file to your current location:

```bash
mv ../sample2_copy.txt .
ls
```

The `mv` command can also be used to rename files:

```bash
mv sample2_copy.txt renamed_sample2_copy.txt
ls
```

Let's get rid of the renamed file. To <ins>r</ins>e<ins>m</ins>ove files, use the `rm` command:

```bash
rm renamed_sample2_copy.txt
ls
```

> [!TIP]
> As a command-line beginne, it might be a good practice to use the `-i` flag with `rm`. This will prompt the terminal to ask for permission before deleting something.

You can also remove entire directories by adding the `-r` flag. The command does this <ins>r</ins>ecursively, i.e., by going into each of the deepest directories, removing their content, moving to the previous directory, removing its content, and so on:

```bash
cd ..
rm -r copies/
ls
```

Summary of useful commands:

| Command                     | Description                                         |
|-----------------------------|-----------------------------------------------------|
| mkdir [dirname]             | Make directory                                      |
| touch [filename]            | Create file                                         |
| rm [filename]               | Remove file                                         |
| rm -i [filename]            | Remove file, but ask before                         |
| rm -r [dirname]             | Remove directory                                    |
| rm *                        | Remove everything in the current folder             |
| cp [filename] [dirname]     | Copy file to another directory                      |
| mv [filename] [dirname]     | Move file to another directory                      |
| mv [dirname1] [dirname2]/   | Move directory to another directory                 |
| mv [filename1] [filename2]  | Rename file                                         |

## 📑 Editing Files

It is often useful to generate new plain-text files quickly using the command line, or make some changes to an existing one. One way to do this is by using a text editor that operates at the command line. Here we will be looking at such a program called `vim`. Let's test it with a file that already exists:

```bash
vim sample1.txt
```

This will open up the vim interface in the viewing mode. To edit a file, you need to change into the <ins>i</ins>nsert mode by pressing the key `i`. Add two sample names:

```bash
sample_A
sample_B
```

To save the file and exit, you need to run a vim command. To do this, first you need to go back to the viewing mode by pressing `esc`. Vim commands always start with `:`. As soon as you press that key, you will see it appearing at the bottom of the screen. You then keep on typing the command you want to run. In this case, you want to <ins>w</ins>rite your edits and <ins>q</ins>uit, so you need to type `:wq` and press `enter`. This should bring you back to the shell interface.

Now try adding "sample_C" and "sample_D" to sample2.txt file.

You can also use vim to create new files. Simply type `vim`, followed by the file name you want and its extension:

```bash
vim mynewfile.txt
```

>[!NOTE]
> The second part of a file name is called *extension*. The extension indicates what type of data the file holds. Here are some common examples:
>
>* `.txt` is a plain text file.
>* `.csv` is a text file with tabular data where each column is separated by a comma.
>* `.tsv` is like a CSV but columns are separated by a tab.
>* `.log` is a text file containing messages produced by a software while it runs.
>* `.pdf` indicates a PDF document.
>* `.png` is a PNG image.
>* `.sh` indicates a shell script.
>  
>While this is a convention, it is a very important one. Files contain bytes: it is up to us and our programs to interpret those bytes according to the rules for plain text files, PDF documents, configuration files, images, and so on.
>Naming a PNG image of a whale as whale.mp3 will not magically convert it into a recording of whalesong, though it might cause the operating system to try to open it with a music player when someone double-clicks it.

To explore the content of a file, we can use the con<ins>cat</ins>enate command `cat`.

```bash
cat sample1.txt
```

Notice that `cat` prints the content of a file to the shell screen. This can be incovenient when we have very large files. To get "sneak-peaks" we can use commands that print smaller chunks instead. For example, we can either use the `head` or the `tail` commands to print the first or last 10 lines (by default), respectively:

```bash
head ../../../../TXST_Microbial-Genomics_2026/exercises/CL1.md
tail ../../../../TXST_Microbial-Genomics_2026/exercises/CL1.md
```

There are a few other options to view files. For example, the command `less` lets you scroll through a file without editing it in a separate interface. Try it:

```bash
less ../../../../TXST_Microbial-Genomics_2026/exercises/CL1.md
```

To exit just press `q`. 

If we wanted to count the number of lines, words, or characters in a file, we can use the <ins>w</ins>ord <ins>c</ins>ount command `wc`:

```bash
wc sample1.txt
```

To *only* get the number of lines, use the argument `-l`:

```bash
wc -l sample1.txt
```

## 🃏 Redirectors and Wildcards
Redirectors change where the output of a command is going. The pipe redirector (`|`), for example, turns the output of a command into the input of another one. If we “pipe” the `ls` command into the `wc -l` command, instead of printing the output from `ls` to the screen as usual, it will go into `wc -l` which will then print out how many items there are in your current working directory:

```bash
ls | wc -l
```

Another important redirector is the greater than sign `>`. This character stores the output of a command into a file, rather than just printing it to the screen:

```bash
ls > directory_contents.txt
cat directory_contents.txt
```

**It is important to remember that the `>` redirector will overwrite any pre-existing file with the name being used.** To append an output to a pre-existing file, rather than overwrite it, we can use `>>` instead.

The concatenate command `cat`, which we have being using to view files, can also be used to concatenate files, as its name suggests:

```bash
cat sample1.txt sample2.txt > combined_1and2.txt
cat combined_1and2.txt
```

Wildcards are special characters that enable us to specify multiple items easily. Let’s say we want to look for files that end with the extension “.txt” in our current working directory. The `*` wildcard can help:

```bash
ls *.txt
```
