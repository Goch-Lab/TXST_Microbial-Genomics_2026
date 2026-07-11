# CL1: Introduction to Command Line

This session, you will get comfortable using the Unix command line, which is an essential skill for working with large datasets, including genomic and other omic data.

---
## 🧠 Learning Objectives

By the end of this computer lab, you should be able to:

- Navigate the file system using `cd`, `ls` and `pwd`
- Create, move, copy, name and delete files and directories
- Use redirectors and wildcards
- Explore files using `head`, `tail`, `less`, `nano` and `cat`.
- Edit text files directly from the shell

---

Let's establish some basics first. 

Linux and Mac users will find a Terminal program already installed on their computers. If you are using Windows, I recommend downloading [MobaXTerm](https://mobaxterm.mobatek.net/), but other software are available. We can also get you set up on your personal computer later if you own a Window/PC. 

Now, the *Terminal* is a text input and output environment where we can execute commands and see the output. In other words, it is the "window" in which you enter the actual commands, and those commands are then interpreted and run by a *Shell*. 

The *Shell*  is the program inside the terminal that actually processes the commands and returns the output. In most Linux and Mac operating systems, it uses a *Bash* shell, which is essentially its own programming language and what we will during this course. 

Different shells provide unique features and syntax: on macOS, the default is Zsh (formerly Bash) with Unix-style commands, while Windows primarily uses PowerShell and Command Prompt, with options to install Unix-like shells such as WSL or Git Bash.

In summary, think of it this way: terminal is the TV and shell is the program running the TV. 


Computers organize file locations in a hierarchical structure. Even though we might not notice, we are typically used to navigating through this stucture by clicking on various folders (also known as directories) in a Mac Finder or Windows Explorer window. Just like we need to select the appropriate files in the appropriate locations in a Graphical User-Interface (GUI), we need to do the same when working at a command-line interface. What this means in practice is that each file and directory has its own “address”, and that address is called a “path”.

Additionally, there are two special locations in all file systems: 🫜"root” and the user’s 🏠“home”. *Root* is where the file system of a computer starts, whereas *home* is where a user’s file system starts (this is where you should be now 📍). An *absolute* path is that one that indicates location based on root; whereas a *relative* path is based on the user's home.

<img width="1511" height="1799" alt="image" src="https://github.com/user-attachments/assets/d051f1d5-91ee-44b7-b175-9471422c174b" />

## Navigating the file system

Before we begin exploring the command, let's get the tutorial files:

```bash
git clone https://github.com/Goch-Lab/TXST_Microbial-Genomics_2026.git
```

Congratulations! 🎉 
You just ran your first command, `git clone`, which specifically allows us to clone files and data from a GitHub reposytory. 

Before we look at some other common commands, note a few keyboard commands that are very helpful:

- `Up Arrow`: shows your last command
- `Down Arrow`: shows your next command
- `Tab`: auto-completes your command
- `Ctrl + L`: clears the screen
- `Ctrl + C`: cancels a command
- `Ctrl + R`: searches for a command
- `Ctrl + D`: exits the terminal
- `history`: shows your command history

Ok, let's explore the other basics.

Let's check where you are by <ins>p</ins>rinting your <ins>w</ins>orking <ins>d</ins>irectory (i.e. where you currently are in the system).
```bash
pwd
```
It should look something like this `/home/[name]` and tell you that you are in the "home" location. 

Now <ins>l</ins>i<ins>s</ins>t the contents of your current directory using the `ls` command.
```bash
ls
```
You should see the directory we just downloaded from Github called "MicrobialGenomics-TXST-2025", we'll explore this in a minute. 

Often, there are also *hidden* files, typically configuration files, which begin with a dot (`.`). E.g., your bash profile is configured by the file ~/.bash_profile. Configuration files do things like store settings and preferences for programs, determine what programs are "turned on" when you log in, and customize how your shell behaves. 

To see these *hidden* files use the **'ls'** *command* with the *argument* -a, to see <ins>a</ins>ll files, including hidden ones.
```bash
ls -a
```
You should see file names like these `.  ..  .bash_history  .bash_logout  .bash_profile  .bashrc  .emacs  example.txt  .kshrc  MicrobialGenomics-TXST-2025  .mozilla  .ssh`. 

We can also use `ls` to see the sizes of the files, in bytes, in our directories with the argument -l.
```bash
ls -l
```

I prefer to add -lh, the "h" prints the sizes in a <ins>h</ins>uman-readable format.
```bash
ls -lh
```

> [!TIP]
> On Linux and Mac, the `man` command is used to show the **manual** of any command that you can run in the terminal. So if you wanted to know more about the `ls` command, you could run:

```bash
  man ls
```

If the `man` command is not included, you can just type the command that you want to know more about and then `--help` and you will get similar info:

```bash
  ls --help
```

You should be able to use the arrow keys or page up and down. When you are ready to exit, just press `q`.


Ok, lets move into our Github downloaded directory, or <ins>c</ins>hange <ins>d</ins>irectories using the `cd` command. Move to the MicrobialGenomics-TXST-2025 directory and use `pwd` and `ls` to see where you are and what is there. **Type each line one at a time and press enter**

```bash
cd MicrobialGenomics-TXST-2025/
pwd
ls
```
What do you see? 

To move back one directory to _home_, simply use `..` like so:
```bash
cd ..
pwd
ls
```
The pwd command should again give you `/home/[name]` and ls should give you `.  ..  .bash_history  .bash_logout  .bash_profile  .bashrc  .emacs  example.txt  .kshrc  MicrobialGenomics-TXST-2025  .mozilla  .ssh`. 

You can also use `cd` to "jump" to other directories quickly, like below:
```bash
cd MicrobialGenomics-TXST-2025/data/01_intro/
pwd
```

If you type `cd` alone, it will bring you all the way back to home. 
```bash
cd
pwd
```

Ok, now let's go back to where we before and try something else.
```bash
cd MicrobialGenomics-TXST-2025/data/01_intro/
pwd
```

When combined with "..", you can move back multiple directories as well.
```bash
cd ../../
pwd
```
Where are you now? (Hint: should be `/home/[name]/MicrobialGenomics-TXST-2025`)

> [!TIP]
> If we are trying to specify a file or path we can begin typing its name and then press the <ins>tab</ins> key to complete it (try it out below). If there is only one possible way to finish what we’ve started typing, it will complete it entirely for us. If there is more than one possible way to finish what we’ve started typing, it will complete as far as it can, and then hitting tab twice quickly will show all the possible options. If tab-complete does not do either of those things, then we are either confused about where we are or what is where, or we've maybe spelled the name wrong.

Let's try this out. Go back to your home with `cd`. Now, let's go back to the `01_intro/` directory, but this time we will use tab to fill out our path, not type it out. Watch first.
```bash
cd MicrobialGenomics-TXST-2025/data/01_intro/
pwd
```

Ok, that was a brief intro to moving around the command line. Practice makes perfect, so practice this when you can, and it will eventually become natural. 

Summary of commands used:
| Command                             | Description                                                                       |
| ----------------------------------- | --------------------------------------------------------------------------------- |
| pwd                                 | Lists the path to the working directory                                           |
| ls                                  | List directory contents                                                           |
| ls -a                               | List contents including hidden files (Files that begin with a dot)                |
| ls -l                               | List contents with more info including permissions (long listing)                 |
| ls -lh                              | List content sizes in readable format                                             |
| cd                                  | Change directory to home                                                          |
| cd [dirname]                        | Change directory to specific directory                                            |
| cd ~                                | Change to home directory                                                          |
| cd ..                               | Change to parent directory                                                        |


## 🧪 Exercise 2: Creating, copying, moving, and removing files + directories
> [!WARNING]
> Using commands that do things like create, copy, and move files at the command line will <ins>**overwrite**</ins> files if they have the same name. And using commands that delete things will do so <ins>**permanently**</ins>. Use caution using these commands.

So we should still be in the `01_intro/` directory. Let's make a new directory in this one. 
Start with <ins>m</ins>a<ins>k</ins>ing a new <ins>dir</ins>ectory, using the `mkdir` command, then move into it.
```bash
mkdir practice
ls
cd practice
```

Now create files in this directory, we will use the command `touch`. Then use `ls` to check that the files are there.
```bash
touch sample1.txt
touch sample2.txt
touch sample3.txt
ls
```

Lets <ins>c</ins>o<ins>p</ins>y the second file and rename it. The first file called is the file you want to copy, the second is what you want to name the copy.
```bash
cp sample2.txt copy_sample2.txt
ls
```

To make a copy and put it somewhere else, like in new subdirectory, we can change the second positional argument using a relative path (“relative” because it starts from where we currently are). Make a new directory called "copies" then copy "Sample 3" there.
```bash
mkdir copies
cp sample3.txt copies/copy_sample3.txt
cd copies/
ls
```

Ok, now we want to move the copy_sample2.txt file here in the copies/ directory. To do that we will use the <ins>m</ins>o<ins>v</ins>e command `mv`), but remember the file is in the previous directory (hint: ".."). 
```bash
mv ../copy_sample2.txt .
ls
```

Here, I am also introducing the use of a single `.`, which is telling `mv` that I want to move the file to where I currently am. 

The `mv` command can also be used to rename files like so:
```bash
mv copy_sample2.txt renamed_copy_sample2.txt
ls
```

Lets get rid of the renamed file. To <ins>r</ins>e<ins>m</ins>ove files, use the `rm` command. 
```bash
rm renamed_copy_sample2.txt
ls
```
> [!TIP]
> When starting out, especially, it might be a better practice to use the -i flag with `rm`. This will prompt the terminal to first ask permission before you delete something.

You can also remove entire directories. Let's remove the copies/ directory using -r. 
```bash
cd ..
rm -r copies/
ls
```
Commands used and other flags
| Command                     | Description                                         |
| --------------------------- | --------------------------------------------------- |
| mkdir [dirname]             | Make directory                                      |
| touch [filename]            | Create file                                         |
| rm [filename]               | Remove file                                         |
| rm -i [filename]            | Remove directory, but ask before                    |
| rm -r [dirname]             | Remove directory                                    |
| rm -rf [dirname]            | Remove directory with contents                      |
| rm ./\*                     | Remove everything in the current folder             |
| cp [filename] [dirname]     | Copy file                                           |
| mv [filename] [dirname]     | Move file                                           |
| mv [dirname] [dirname]      | Move directory                                      |
| mv [filename] [filename]    | Rename file or folder                               |
| mv [filename] [filename] -v | Rename Verbose - print source/destination directory |

## 🧪 Exercise 3: Editing/creating file contents

It is often very useful to be able to generate new plain-text files quickly at the command line, or make some changes to an existing one. One way to do this is using a text editor that operates at the command line. Here we’re going to look at one program that does this called `nano`. Let's test it with a file that already exists.
```bash
nano sample1.txt
```

This will open up an interface and allow you to add text. Add two sample names, A and B.
```bash
sample_A
sample_B
```

To save the file and exit, we need to use some of the keyboard shortcuts listed on the bottom. Type "ctrl" + "x". It will ask if you want to save, type "y" and then press "enter". Alternatively, if you wanted to change the file name, you can before pressing enter. 

Now try adding "sample_C" and "sample_D" to sample2.txt file.


I also use nano to create new files, you just simply type `nano`, followed by the file name you want and its extension. E.g.:
```bash
nano mynewfile.txt
```

>[!NOTE]
> The second part of a file name is called the filename extension, and indicates what type of data the file holds. Here are some common examples:
>* .txt is a plain text file.
>* .csv is a text file with tabular data where each column is separated by a comma.
>* .tsv is like a CSV but values are separated by a tab.
>* .log is a text file containing messages produced by a software while it runs.
>* .pdf indicates a PDF document.
>* .png is a PNG image.
>* .sh indicates a shell script file.
>  
>This is just a convention, albeit an important one. Files contain bytes: it’s up to us and our programs to interpret those bytes according to the rules for plain text files, PDF documents, configuration files, images, and so on.
>Naming a PNG image of a whale as whale.mp3 doesn’t somehow magically turn it into a recording of whalesong, though it might cause the operating system to try to open it with a music player when someone double-clicks it.

Ok, back to viewing files.

To get a "sneak-peak" at what we added in the sample1.txt file, we can either use the `head` command to show the top of the file contents. There is also `tail`, which prints the last 10 lines of a file by default:
```bash
head sample1.txt
```
There are a few other options to view files.
For example, the command `less` lets you scroll through a file but not edit it. Try it out on `sample1.txt`. (To exit the `less` command, just press `q`. 

The command `cat` will display the whole file at once, so it is better to use for shorter files.
Try it out too `sample1.txt` file.

If we wanted to count the number of lines, words, or characters a file has, we can use the `wc` or (<ins>w</ins>ord <ins>c</ins>ount) command. 
```bash
wc sample1.txt
```

To *only* get the number of lines in the file, use the argument -l.
```bash
wc -l sample1.txt
```
How many lines are there? 


## 🧪 Exercise 5: Redirectors and Wildcards

Redirectors change where the output of a command is going. Lets look at an example using a "pipe" (`|`).
Remember our commands `ls` and `wc -l`? If we “pipe” (`|`) the `ls` command into the `wc -l` command, instead of printing the output from `ls` to the screen as usual, it will go into `wc -l` which will print out how many items there are in your current working directory:
```bash
ls | wc -l
```

Another important character is the greater than sign, `>`. This tells the command line to redirect the output to a file, rather than just printing it to the screen as we’ve seen so far. For an example of this we will write the output of ls to a new file called “directory_contents.txt”:
```bash
ls > directory_contents.txt
head directory_contents.txt
```
**It’s important to remember that the `>` redirector will overwrite the file we are pointing to if it already exists.**
If we want to append output to a file, rather than overwrite it, we can use two signs, `>>` instead.

Now the `cat` command, which we used above to view a file, also serves the purpose of combining (conCATenate) the content of files together. Let's see what happens when we concatenate sample1.txt and sample2.txt. 
```bash
cat sample1.txt sample2.txt >combined_1and2.txt
head combined_1and2.txt
```

Now, wildcards are special characters that enable us to specify multiple items very easily. Let’s say we only want to look for files, in our current working directory, that end with the extension “.txt”. The * wildcard can help us with that.
```bash
ls *.txt
```
What this is saying is that no matter what comes before, if it ends with “.txt” we want it.


## 🧪 Exercise 4: Terminus game

Practice navigating the command line by making your way through the world of Terminus. The locations you can go to are directories, which you `cd` to move back and forth, and `ls` to look around and see what is in the area. Items in each area are files, so you use `less` to open them (interact with them), or `cp` to make copies of an "item," `touch` to create an item you need, `rm` to remove an obstacle, etc. If you need help with a command, type `man` and the command you want help with. 

Follow this link to access Terminus: https://web.mit.edu/mprat/Public/web/Terminus/Web/main.html 
When it loads, follow the instructions. Have fun!

📝 ## Assignment Due before
Turn in the answers to the questions below on Canvas before next Thursday's class (1:59 pm 09/04). 

Send me three screenshots:
1. When you had to use the command `mv`. 
2. When you had to use the command `rm`. 
3. When you had to use the command `cp`. 




