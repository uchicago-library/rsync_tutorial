# About these notes

The purpose of these notes is to summarize how the Preservation department might use [rsync](https://en.wikipedia.org/wiki/Rsync) to copy and move large amounts of data between servers quickly and securely.

Rsync is a command-line program. Using this program to copy files helps to prevent some common errors, like accidentally moving a large number of files into the wrong location by accidentally releasing the mouse button too early during a drag and drop. It's also a very powerful command, with options for many different scenarios. The cost for that power is that rsync has a bit of a learning curve. Please feel free to reach out with any questions or concerns about how the program works. 

# Understanding rsync

Rsync (pronounced with two syllables- the letter "r", followed by "sync", as in "synchronize") is a program that syncs two directories. But what does that mean?

Imagine a directory tree on your local machine with thousands of files organized into hundreds of subdirectories. Lets say there are 10Gb of files in all, and you copied all of those directories and files to a remote server. But then, you realized that ten of the files needed to be updated. You do your updates. 

Now, it would be possible to copy each of those ten files to the remote server, using ten separate `scp` (secure copy) commands. However, in doing that it would be easy to miss a file, or to accidentally copy a single file to the wrong location- and it would also be very difficult to figure out that this had happened. 

Rsync solves this problem. Instead of copying files one at a time, you'll tell rsync to recursively synchronize two entire directory trees. Rsync will look at each file in a given directory tree on your local machine (the source directory) and another on a remote server (the target directory), and it will compare each of the files it finds. When it finds a file that differs, it will copy just the differences (the "deltas".) In our example, the files on your local machine were the definitive ones, and we wanted to copy any changes to the remote server. However, you could also use rsync to copy from the remote server to your local machine, in cases where the definitive files are remote. In this case the remote directory would be the source and your local directory would be the target. If you delete a file from the source directory you can tell rsync to delete it from the target, or to keep it. 

# Preliminary setup

I assume you'll be using a Windows machine. If so you'll need to install the [Windows Subsystem for Linux (WSL)](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux), using a command like the following:

```console
wsl --install
```

By default this will install [Ubuntu Linux](https://ubuntu.com/) on your machine. After the installation, Ubuntu will prompt you for a local username and password. For the username you can use your CNetID, but please choose a different password. You'll need to use this password whenever you use the `sudo` command to run something as the root user. 

From a Windows terminal, enter `wsl` to start a WSL terminal. 

You may want to install the [Windows Terminal](https://apps.microsoft.com/store/detail/windows-terminal/9N0DX20HK701?hl=en-us&gl=us) from the Microsoft Store for a more comfortable experience using the command line. 

# Creating an SSH key pair

To upload to staff.lib you'll need to create an SSH key pair. From Ubuntu, enter the following command, replacing `user` with your CNetID. 

```console
ssh-keygen -t rsa
ssh-copy-id user@staff.lib.uchicago.edu
```

# Getting familiar with the command prompt (a.k.a., "the terminal")

If you'd like a tutorial about switching from a graphical user interface (GUI) to the command line interface (CLI), there are lots online to choose from. I like [Navigating The Linux Terminal (pwd, ls & cd) - The Basics!](https://www.youtube.com/watch?v=P0KeDt-GuEI) from NCSU BIT. Although it uses a slightly different setup than what you'll be using, I think the narrator does a really great job of explaining how to make the switch from using a GUI to a CLI.

# Finding directories on your local machine

Your local drives are mounted under `/mnt/`. Use commands like the following to look around:

```console
cd /mnt
ls
cd d
pwd
```

The last command, `pwd`, prints the full path to the current directory on your local machine. Make a note of the exact path- you'll need it when you run rsync. 

# Finding directories on the remote server

From inside Ubuntu, use the following command to open a secure shell (SSH) to pressync.lib:

```console
ssh pressync.lib.uchicago.edu
```

Folders for Preservation have been mounted under `/data`. Try this to view them, and to cd into `pres-xfer`.

```console
cd /data/
ls
cd pres-xfer
pwd
```

And again, `pwd` prints the current directory, only this time it does it on the remote server. Make a note of this path for rsync too- it will be something like `/data/pres-xfer/mvol/`.

# rsync cheatsheet

You'll use a command like this to actually sync the files:

rsync -rltvz /src/local-dir/ pressync.lib.uchicago.edu:/data/pres-xfer/mvol

# Keeping Ubuntu and WSL up to date

Run the following command from the WSL terminal to periodically update your Ubuntu installation:

```console
sudo apt-get update && sudo apt-get upgrade
```

Run the following command from the Windows terminal to periodically update WSL itself:

```console
wsl --update
```

# Errors and improvements

Please feel free to reach out to jej@uchicago.edu with suggestions for how to improve these notes. 
