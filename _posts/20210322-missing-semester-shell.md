---
title: Missing Semesters (2020) 1강 The Shell
date: 2021-03-22 20:38:19
mathjax: true
tags: 
- Missing Semesters
- MIT
- Shell Script
- Bash
categories: 
- [  SW, MIT Missing Semesters 정리]
excerpt: Missing Semesters (2020) - The Shell 정리
---

{% youtube Z56Jmr9Z34Q %}

&nbsp;

## Shell

- One of the primary ways of interacting with the computer
- Visual interfaces are limited
  - You can only use the functions that the buttons provide.
- Most platforms provide some sort of shells
  - Windows - Powershell
  - Linux - Bash shell
  - MacOS - Bash shell

&nbsp;

## Shell prompt

- We can customise our shell prompts
- We get to write shell commands on the shell prompts
  - We can execute a program with arugments
  - Examples:
    - `date` prints today's date
    - `echo` prints out the arguments given
      - arguments are something that's separated by a blank space
        - `echo hello`.
        - `echo "Hello world"`
        - `echo Hello\ world`
- How does the shell know what these programs are?
  - Your OS comes with some built-in programs, stored on your file system.
  - **Environment variable**
    - Things that are set when we start the terminal.
      - e.g. Where is the home directory? What is my user name?
    - **Path variable**
      - `echo $Path` will show all the directories that the shell will search for programs.
      - When we try to run a program on terminal, bash will search through all the paths stated in the path variables for the file/program that matches with the program we typed onto the terminal.
        - e.g. Bash will search for a file called `echo` throughout all paths stored in the path variable.
        - If I'm not sure which `echo` I am running, then I can use `which echo` command to find out what program I am running.

&nbsp;

## Navigating file systems + using programs

- UNIX systems use `/` as the root directory. Windows doesn't.
- Use `pwd` to print my working directory.
- Use **`cd` to change my current working directory.**
  - We can use relative paths to navigate around system
  - `cd /` - Absolute path for root directory
  - `cd ..`. `cd ./home`, `cd ../../../../../`
  - Use `cd ~` to go to home directory (i.e. `/home/username/`).
  - Use `cd -` to go back to the directory you were at.
- Use **`ls` to see what files and directories are in the current working directory.**
- Use `program_name --help` to print out what flags and arguments we can use for the program.
  - `[]` parameters are optional parameters
  - `...` means multiple of them.
- `man program_name` will show the program manual
- `mv` will move files
- `cp` will copy files
- `rm` will remove files
  - `rm -r` will remove file/directory in recursive manner
  - `rmdir` will remove an empty directory

&nbsp;

## Permissions

- Use `ls -l` to view files and directories in long listing format.
  - The first few letters tell us about many things
  - If the first letter is `d`, it's a directory. Otherwise it's a file.
  - The rest of the letters mean the permissions granted for each user groups:
    - The owner of the file
    - The group that owns this file
    - Anyone else
  - **`r` - read permission**
  - **`w` - write permission**
  - **`x` - execute permission**
  - `-` - Don't have permission

&nbsp;

## Streams

- Every program has 2 primary streams by default
  - Input stream
  - Output stream
- **`<` or `>` will rewire the streams**
  - `<` rewire the input of the program to be the contents of a file
  - `>` rewire the output of the program into a file.
  - e.g. `echo hello > hello.txt` will save 'hello' to hello.txt. Nothing will print, but the content of hello.txt will be hello. We can check this by `cat hello.txt`. `cat` will print the contents of a file.
  - We can also use `cat < hello.txt`
  - We can copy contents of a file without using `cp`, by doing `cat < hello.txt > hello2.txt`. This will print the contents of 'hello.txt' and output it to 'hello2.txt'.
- `>>` will append the output.
  - `cat < hello.txt > hello2.txt`, then `cat < hello.txt >> hello2.txt` will print `hello\n hello`.
- We can use `|` to make more sophisticated streams.
  - `|` will take the output of the left side's program, to be the input of the right side's program.
  - e.g. `ls -l / | tail -n1`. `ls -l /` will print the files in root directory in long format. `tail -n1` will read data and output the last line. Overall, this code will output the last line of the `ls -l /`.
  - e.g. `curl --head --silent google.com | grep --ignore-case content-length | cut --delimiter=' ' -f2`. The first part will grab the information of 'google.com'. The second part uses `grep` command to extract the information with the header 'content-length'. The third part then cuts the string by a space, and then outputs the second field (which is the value of the content-length of google.com). 

&nbsp;

## Root user mode

- Use **`sudo` to activate superuser mode**.
  - 'do as superuser'.
  - You don't want to run as superuse all the time. Often critical failures mas superuse may lead to complete system failure.
- Use `cd /sys`
  - These files are not actual files in your system.
  - These are various kernels parameters (i.e. the core of your computer)
  - e.g. `cd /sys/backlight/intel_backlight/`, `cat brightness` to show the current brightness of the screen. We can use `sudo su` to activate root user mode, and then use `sudo echo 500 > brightness` to modify the brightness.
    - **When running as a superuser (i.e. root user), we will see `#` on the terminal.**
      - We use `exit` to exit the root user mode.
    - **When not running as a super user, we see `$` on the terminal.**
  - Running `echo 1000 | sudo tee brightness` will work without being a root user. The `tee` command will write the input data to a file, but also will print the data at the same time.

&nbsp;

## How to open a file

- Use `xdg-open` to open a file in Linux. This will use an appropriate program to open a file.