# A short rsync tutorial

## About this document

This tutorial shows how to use [rsync](https://en.wikipedia.org/wiki/Rsync) to copy large amounts of data between servers quickly and securely. Although rsync is a commandline program, these notes try to avoid assuming any experience with the command line. 

Rsync helps to prevent some common errors that happen when you copy a large nubmer of files, like accidentally files to the wrong place by releasing the mouse button too early during a drag and drop. It is also very reliable and fault-tolerant. It's a powerful command, which comes with a bit of a learning curve. Hopefully these notes will help you get started. Please feel free to reach out with any questions or concerns so we can continue to improve them.

## What does rsync do?

Rsync (pronounced "ARR-sink") is a program to synchronize two directories, a source and a target. But what does that mean?

Imagine a directory tree on your local machine with thousands of files organized into hundreds of subdirectories. Lets say there are 10Gb of files in all, and you copied all of those directories and files to a remote server. But then, you realized that ten of the files needed to be updated. You do your updates on your local machine. 

Now, it would be possible to copy each of those ten files from your local machine to the remote server using ten separate `scp` (secure copy) commands. However, in doing that it would be easy to miss a file, or to accidentally copy a single file to the wrong location. It would be difficult to figure out that this had happened. 

Rsync was designed to solve this problem by working with whole directory trees- all of the files and subdirectories that exist inside some parent directory. Instead of copying files one at a time, rsync looks at each file in a directory tree on your local machine and another on a remote server, and it will compare each of the files it finds. When it finds a file that differs, it will copy just the differences, which makes it very fast.

In this example, the files on your local machine were the definitive ones, and we wanted to copy any changes to the remote server. This means that the source directory was local, and the target directory was remote. Rsync is a good tool to use when you can say that one of the two directories involved is the definitive one, or the source, and that the target directory and its contents can be made to match the source. In situations where edits have been made to files in both directories I would choose different tools. 

If the remote server contained the definitive set of files, you could call it the source and your local directory the target, and in that case rsync would make your local directory match what it sees on the remote server. The source and target could both be local, or they could both be remote. The important thing to remember is that rsync makes the target directory match what it finds in the source. 

However, you could also use rsync to do the opposite, in cases where the definitive files are remote. In that case the remote directory would be the source and your local directory would be the target. If you delete a file from the source directory you can tell rsync to delete it from the target, or to keep it. Rsync has a lot of options- if you're curious about how to use it in a new situation, just ask.

## Preliminary setup

I assume you'll be using a Windows machine. If so you'll need to install the [Windows Subsystem for Linux (WSL)](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux), using a command like the following:

```console
wsl --install
```

By default this will install [Ubuntu Linux](https://ubuntu.com/) on your machine. After the installation, Ubuntu will prompt you for a local username and password. For the username you can use your CNetID, but please choose a different password. You'll need to use this password whenever you use the `sudo` command to run something as the root user. 

From a Windows terminal, enter `wsl` to start a WSL terminal. 

You may want to install the [Windows Terminal](https://apps.microsoft.com/store/detail/windows-terminal/9N0DX20HK701?hl=en-us&gl=us) from the Microsoft Store for a more comfortable experience using the command line. 

## Creating an SSH key pair

To upload to staff.lib you'll need to create an SSH key pair. From Ubuntu, enter the following command, replacing `user` with your CNetID. 

```console
ssh-keygen -t rsa
ssh-copy-id user@staff.lib.uchicago.edu
```

## Getting familiar with the command prompt (a.k.a., "the terminal")

If you'd like a tutorial about switching from a graphical user interface (GUI) to the command line interface (CLI), there are lots online to choose from. I like [Navigating The Linux Terminal (pwd, ls & cd) - The Basics!](https://www.youtube.com/watch?v=P0KeDt-GuEI) from NCSU BIT. Although it uses a slightly different setup than what you'll be using, I think the narrator does a really great job of explaining how to make the switch from using a GUI to a CLI.

Before moving on, you should know how to use `cd` to change directories, `ls` to list the contents of directories, `pwd` to see the full path the the current directory. It would be helpful to know how to use `cp` to copy files, and `mv` to move them as well.  

## Finding directories on your local machine

Your local drives are mounted under `/mnt/`. Use commands like the following to look around:

```console
cd /mnt
ls
cd d
pwd
```

To use rsync you'll want to find the absolute path to the source files on your local machine. Absolute paths begin with a slash (/)- they make very clear exactly which diretory you'll be syncing from. Make a note of the path to your source directory. It will look something like `/mnt/c/digital_files/`.

## Finding directories on the remote server

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

Find the absolute path to your target directory and make a note of it. It will be something like `/data/pres-xfer/mvol/`.

## Rsync cheatsheet

### Make the contents of a remote directory match a local one

```console
rsync -rltvz /mnt/c/src/rsynctest/ pressync.lib.uchicago.edu:/data/pres-xfer/rsynctest/
```

We specified a few options above- here are some notes on each one:

-r, or --recursive. This tells rsync to "recurse" into subdirectories. 

-l, or --links. This tells rsync to copy symbolic links, or symlinks, as symlinks. 

-t, or --times. Rsync will preserve the timestamps of each file it copies. 

-v, or --verbose. Rsync will report more of what it is doing to the user. 

-z, or --compress. This option tells rsync to compress the data it sends, which shortens sync time. 

Note that in this command the source directory (/mnt/c/src/rsynctest/) comes first and the target directory (pressync.lib.uchicago.edu:/data/pres-xfer/rsynctest/.

Also note the trailing slash at the end of the source directory. This means, "copy the contents of this directory". Omitting that trailing slash would have meant "copy this directory". The difference is subtle but important- here, I want `pressync.lib.uchicago.edu:/data/pres-xfer/mvol/` to match the local directory `/mnt/c/src/rsynctest/`. I do not want to create a second mvol directory in the target at `pressync.lib.uchicago.edu:/data/pres-xfer/rsynctest/rsynctest/`. 

### Delete files in the target directory that have been deleted in the source

The command above copies files that exist on the source to the target, but it does not delete files from the target if they had been deleted on the source. That is a more dangerous operation, but here is how you can do that. First, use the `--dry-run` option to see what rsync is going to do without actually doing it yet:

```console
rsync -rltvz --delete --dry-run /mnt/c/src/rsynctest/ pressync.lib.uchicago.edu:/data/pres-xfer/rsynctest/
```

You'll see a list of deletions in the output rsync provides. If that looks good, go ahead and run the command by omitting `--dry-run`:

```console
rsync -rltvz --delete /mnt/c/src/rsynctest/ pressync.lib.uchicago.edu:/data/pres-xfer/rsynctest/
```

### Output results to a log file

Log files provide a receipt you can use to prove that you copied specific files.

```console
rsync -rltvz --log-file=20230630.log /mnt/c/src/rsynctest/ pressync.lib.uchicago.edu:/data/pres-xfer/rsynctest/
```

You can name your log file whatever you like, but in the example above, I used the date 20230630 to show that I produced the log file on June 30, 2023. 

### Checking to be sure files were copied correctly

Because rsync checks files to see if there are any differences before syncing, you can invoke the command a second time to confirm that all files were successfully transferred. 

```console
rsync -rltvz --log-file=20230630.2.log /mnt/c/src/rsynctest/ pressync.lib.uchicago.edu:/data/pres-xfer/rsynctest/
```

Here I name the log files "20230630.2.log" to store two separate logs- the first to show which files were copied, and the second to prove that they were copied successfully. 

### Removing files from the source directory after copying

```console
rsync -rltvz --remove-source-files /mnt/c/src/mvol/ pressync.lib.uchicago.edu:/data/pres-xfer/rsynctest/
rm /mnt/c/src/rsynctest/
```

Here rsync will bascially move files instead the source directory instead of just copying them. The second command, `rm /mnt/c/src/rsynctest/`, removes the outer `mvol` directory itself. 

## Using man pages to learn more

Most command line programs come with man (short for "manual") pages that explain their options in detail. From inside Ubuntu, use the following command to see rsync's man page:

```console
man rsync
```

## Keeping Ubuntu and WSL up to date

Run the following command from the WSL terminal to periodically update your Ubuntu installation:

```console
sudo apt-get update && sudo apt-get upgrade
```

Run the following command from the Windows terminal to periodically update WSL itself:

```console
wsl --update
```

## Notice

Rsync tutorial, some notes about getting started with rsync. 

Copyright (C)

This program is free software; you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation; either version 2 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program; if not, write to the Free Software Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA 02111-1307 USA 

## Errors and improvements

Please feel free to reach out to jej@uchicago.edu with suggestions for how to improve these notes.
