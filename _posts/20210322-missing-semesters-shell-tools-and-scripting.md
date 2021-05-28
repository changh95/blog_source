---
title: Missing Semesters (2020) 2강 Shell Tools and Scripting
date: 2021-03-22 22:38:19
mathjax: true
tags: 
- Missing Semesters
- MIT
- Shell Script
- Bash
categories: 
- [  SW, MIT Missing Semesters 정리]
excerpt: Missing Semesters (2020) - Shell Tools and Scripting 정리
---

{% youtube kgII-YWo3Zw %}

&nbsp;

## Variables

- **`foo=bar` to assign a variable**
- **`echo $foo` to access and print the variable**
- **Spaces are important** - we separate arguments with spaces.
  - i.e. `foo = bar` does not work.
- Strings?
  - `echo "Hello"` and `echo 'World'`
  - The difference in double quotes and single quotes are...
    - `echo "Value is $foo"` prints 'Value is bar'
    - `echo 'Value is $foo'` prints 'Value is $foo'

&nbsp;

## Functions

- We can make funcions by `.sh` files.
  - e.g. Inside the `mcd.sh` file, we have...

```bash
mcd() {
    mkdir -p "$1"
    cd "$1"
}
```

- This function is like mv + cd.
- **`$1` is a special variable, similar to argv.**
  - It means the first argument.
  - Similarily, `$2`, `$3`... they will be second and third arguments, up to 9th.
  - `$0` is reserved for the name of the script.
- We can use **`source mcd.sh` to load the function into the shell.**
  - We can now use `mcd test` command to create a directory called test and move into it.

&nbsp;

## Special commands

- `!!` gives the last command given to the shell.
  - If I used `mkdir ~/Downloads/folder1` in the previous comment, I can then use `sudo !!` to represent `sudo mkdir ~/Downloads/folder1`.
- `$_` gives the last argument from the previous command... like below.

```bash
rmdir test
mkdir $_
cd $_
```

&nbsp;

## Using error codes

- **`$?` gives the error code** from the previous command.
  - `echo "Hello"`, then `echo $?` will print 0, since there is no error (i.e. error code 0).
  - `grep foobar mdc.sh`, then `echo $?` will print 1. This is because there is no string 'foobar' in the mcd.sh, so the error code 1 is given.
  - `true`, then `echo $?` will always have error code 0.
  - `false`, then `echo $?` will always have error code 1.
  - **We can use this like `false || echo "Oops fail"`. Bash will run the first command, and if it fails, then we will run the second command.**
    - i.e. `true || echo "This will not be printed"`.
  - **We can also use this `true && echo "This works well"`. Bash will run the first command, and if it suceeds, then we will run the second command.**
- **We can use `;` to run two functions and concatenate the results too.**
  - e.g. `false ; echo "This will print anyways"`.

&nbsp;

## Using functions and variables together

- `foo=$(pwd)` will store the results of pwd into foo.
  - We can access it by `echo $foo`
  - We can also use it like `echo "We are in $(pwd)"`
- We can concatenate the outputs of two functions like
  - `cat <(ls) <(ls ..)`.
  - We execute the function, and then get the output as a temporary file to feed into the function next to it.

&nbsp;

## Example function

```bash

echo "Staring program at $(date)" # Date will be substituted

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    # When pattern is not found, grep has exit status 1
    # We redirect STDOUT and STDERR to a null register since we do not care about them

    ,if [[ :$?" -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done

```

- `date` is a built-in bash function.
- `$0` is the name of the script we are running
- `$#` is the number of arguments we are giving to this program.
- `$$` is the process ID of this command.
- `$@` puts all the arguments.
  - If we do not know how many arguments are there ($1, $2, $3...), then we can use `$@` to put all the arguments.
- In a for loop, we make the arguments into `file` variable. (i.e. We are running a loop over every argument given. Maybe the arguments are file names)
- `grep foobar "$file"` means we are going to try to find 'foobar' in the file (i.e. arguments)
- `>` is sending the output of a program to a file.
- `/dev/null` is a special space in UNIX that gets discarded regardless of how much we put it in.
- `2>` is sending the STDERR of a program to a file.
- `$?` gives the error code of the previous command.
- `-ne` is a comparison operator of 'not equal'. (Similar to !=)
- `"$?" -ne 0` will try to see if the error code is 0 (i.e. foobar was not found)
  - Then, it will copy a comment of `# foobar` into the file.
- `fi` is maybe the end of if statement?
- `done` is maybe the end of the script? 

&nbsp;

## Globbing

- `*.sh` **Globbing** all files with the same extension '.sh'.
- Say there are 'project1', 'project2', 'project42' in the directory. 
  - Using `ls project?` will give 'project1' and 'project2' as it only suggests possibilities with 1 single character.
- **Curly braces **are power tools.
  - When running `convert image.png image.jpg`, we can do it as `convert image.{png, jpg}`
  - `touch foo{,1,2,10}` will mean `touch foo, foo1, foo2, foo10`.
  - `touch project{1,2}/src/test/test{1,2,3}.py` will be very useful in automation.
  - `mkrdir foo bar`, then `touch {foo,bar}/{a...j}` will be useful as well.

&nbsp;

## Shell tricks - Python

```python
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```

- The first line `#!/usr/local/bin/python` will allow the user to run the script like `./scripy.py a1 a2 a3` instead of the normal way `python3 ./script.py a1 a2 a3`.
  - The downside of this method is that different environments might have different directories of saving python.
  - So in this case, **`#!/usr/bin/env python` is actually more useful.**

&nbsp;

## Debugging shells

- `shellcheck example.sh`.

&nbsp;

## find

- `find . -name src -type d` will recursively go through the current directory to find a directory called 'src'.
- `find . -path '**/test/*.py' -type f` will recursively go through the current directory to find a python file under the test folder.
- `find . -mtime -1` will find the files that has been 'modified'(i.e. m-times) by 1 time.
- `find . -name "*.tmp" -exec rm {} \;` will find all the files with '.tmp' extension and remove them.
  - On the shell, it will look like nothing happened, but when you check with `echo $?`, you will find that it had exit code 0.

&nbsp;

## locate

- `locate some_string` will do something similar to find.
  - However, locate is much faster as it searches through the index that the UNIX system built already (it is very similar to database approach), so it's much faster than brute-force search method of 'find'.
- `updatedb` command will update this database.

&nbsp;

## grep

- `grep foobar example.sh` will search for the contents of the file. 
  - This is different from 'find' or 'locate', as these are for directories.
- `grep -R foobar .` will recursively search for the string amongst all of the files in the directory.

&nbsp;

## rg - ripgrep

- ripgrep is an installed package - `sudo apt install ripgrep`
  - Just a better alternative to 'grep', as it has colour coding, unicode supports, fast recursive search with respect to gitignore etc.
- `rg "import requests" -t py -C 5 ~/scratch` will find all the python files that uses 'import reuqests'. 
- `rg -u --files-without-match "^#\!" -t sh`
  - `-u` means don't ignore hidden files
  - `--files-without-match` means I want to print the files that do NOT match with the pattern. (This is quite hard with grep)
  - `"^#\!"` is a regex saying that the beginning of a line has a '#' and '!'.

&nbsp;

## Searching command history

- `history` will print the recent commands
- `history 1` will print all the commands from the beginning of the time.
- `history 1 | grep convert` will print the times when we used 'convert' function.

&nbsp;

## Navigation

- `tree` builds a directory tree easily