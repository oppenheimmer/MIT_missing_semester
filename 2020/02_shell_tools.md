# Scripting & Shell tools

## Elementary scripting

We have learned how to execute individual commands and connect them using pipelines. However, in many situations you will need to run a sequence of commands and use control flow constructs such as `conditionals` and `loops`. You may also need to define a `function` to reuse blocks of commands.

Shell scripts are the answer. Most shells provide their own scripting syntax, including support for *variables* and *control flow*. What distinguishes shell scripting from general-purpose programming languages is that it is specifically designed for the shell and the operating system. Creating command sequences, redirecting output to files, and reading from standard input are fundamental operations in shell scripts. As a result, shell scripting is often convenient for automating command-line workflows.

## Variables

To assign variables some value, consider this example

```console
~$ foo=bar 
~$ echo $foo
   bar
```

We can access a variable’s value using `$` (for example, `$varname`). Note that `foo = bar` will not work because of the whitespace around `=`. The shell interprets this as a command named `foo` with `=` and `bar` as arguments, rather than as a variable assignment. Strings can be defined using either single quotes (`'`) or double quotes (`"`), but they are not equivalent. Strings enclosed in single quotes are treated as *literal strings* and do not perform variable or command substitution. In contrast, strings enclosed in double quotes allow *variable expansion* and other substitutions, so variables and certain tokens are evaluated when the string is used.

```console
~$ foo=bar
~$ echo "$foo" # prints bar
   bar
~$ echo '$foo' # prints $foo
   $foo
```

As with most programming languages, `bash` supports control flow constructs such as `if`, `case`, `while` and `for`. It also supports functions that accept arguments and operate on them. Here is an example:

```bash
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```

`$1` refers to the first positional argument passed to a function or script. Unlike many other scripting languages, Bash uses a set of special parameters to reference arguments, exit statuses, and other execution-related values. Below is a brief list:

- `$0`      - Script name  
- `$1`–`$9` - Positional parameters passed to the script. `$1` is the first argument, `$2` the second, and so on (up to `$9`).  
- `$@`      - All positional parameters, treated as separate quoted arguments (`"$@"`).  
- `$#`      - Number of positional parameters passed to the script.  
- `$?`      - Exit status of the most recently executed command.  
- `$$`      - Process ID (PID) of the current shell or script.  
- `!!`      - The entire previous command, including its arguments (history expansion, interactive shells).  
- `$_`      - The last argument of the previous command.  


User-defined functions must either be *sourced* into the current shell session or placed in a directory included in `$PATH` to be accessible as commands. Commands typically write normal output to `STDOUT`, error messages to `STDERR`, and provide an integer return value to indicate execution status. This *exit status* or *return code* is how commands communicate success or failure. An exit status of `0` represents success, while any nonzero value represents failure. Conditional execution (&&, ||, if) depends on the numeric exit status of commands. In shell semantics, `true` returns `0`, and `false` returns `1`. Exit statuses can be used to control conditional execution with `&&` (AND) and `||` (OR). These are short-circuit operators:

* For the `OR` operator (`||`), the second command executes only if the first command fails (nonzero exit status).
* For the `AND` operator (`&&`), the second command executes only if the first command succeeds (exit status `0`).

Commands can also be separated on the same line using a semicolon (`;`). Let's see some examples.


```bash
$ false || echo "Oops, fail"
# prints Oops, fail

$ true || echo "Will not be printed"
#

$ true && echo "Things went well"
# prints Things went well

$ false && echo "Will not be printed"
#

$ true ; echo "This will always run"
# prints This will always run

$ false ; echo "This will always run"
# prints This will always run
```

```bash
#!/bin/bash

echo "Starting program at $(date)"  # Date will be substituted
echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    # When pattern foobar not found, grep has exit status 1
    # We redirect STDOUT and STDERR to a null register since we do 
    # not care about them

    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```



## Command and Process substitution

Sometimes we require the output of a command as an input to another command or assign it to a variable. This can be done with *command substitution*. Whenever you place `$(CMD)`, it will execute `CMD`, capture its output, and substitute it in place. For example, consider this snippet:

```console
~$ echo "We are in $(pwd)" # Double quotes expand values within
```

A lesser-known similar feature is *process substitution*. <(CMD) executes CMD and exposes its output as a file-like object (often a named pipe or `/dev/fd` entry), whose path replaces the `<(...)` expression. This is useful when commands expect input to be passed by file instead of by standard input `STDIN`. For example, `diff <(ls .) <(ls ..)` will show the differences between files in the current directory and its parent.

> * `Command substitution`: The shell executes a command and replaces the substitution expression with the command’s output.
> * `Process substitution`: The shell executes a command and exposes its output as a file-like object (typically a named pipe or `/dev/fd` entry), whose path is substituted into the command line.


### Wildcard and Brace expansion

When launching scripts, we often require arguments that follow a similar pattern. `bash` provides mechanisms to simplify this by expanding expressions through filename expansion. These techniques are commonly referred to as shell *globbing*.

* **Wildcard**: To perform wildcard matching, you can use `?` and `*` to match exactly one character and zero or more characters, respectively. For instance, given the files `foo`, `foo1`, `foo2`, `foo10`, and `bar`, the command `rm foo?` will delete `foo1` and `foo2`, whereas `rm foo*` will delete `foo`, `foo1`, `foo2`, and `foo10`, but not `bar`.

* **Brace expansion**:  When you have a common substring in a series of filenames or commands, you can use curly braces to expand this automatically. For example, `mv file{1,2,3}.txt backup/` expands to `mv file1.txt file2.txt file3.txt backup/`. This is particularly useful when moving or converting multiple files with predictable names.


```bash
convert image.{png,jpg}
# Will expand to
# convert image.png image.jpg

cp /path/to/project/{foo,bar}.sh /newpath
# Will expand to
# cp /path/to/project/foo.sh /newpath
# cp /path/to/project/bar.sh /newpath

# Globbing techniques can also be combined
mv *{.py,.sh} folder/
# Will move all *.py and *.sh files to folder

mkdir foo bar
touch {foo,bar}/{a..h}
touch foo/x bar/y
# This creates files foo/a, foo/b, ... foo/h, bar/a, bar/b, ... bar/h
# and then x under foo, and y under bar

# Show differences between files in foo and bar
diff <(ls foo) <(ls bar)
```

## Scripting Tools

Writing `bash` scripts can be tricky and unintuitive. There are tools like [shellcheck](https://github.com/koalaman/shellcheck) that will help you find errors in your shell scripts. Note that scripts do not necessarily need to be written in Bash to be executed from the terminal. For instance, here is a simple Python script that outputs its arguments in reverse order:

```python
#!/usr/bin/env python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```

The kernel uses the shebang to determine which interpreter to invoke. It does not "know" in a semantic sense; it parses the first line. It is good practice to write shebang lines using the `env` command, which resolves the interpreter’s location and increases portability. To determine the correct executable, `env` uses the `PATH` environment variable rather than a hardcoded path. Depending on the current environment, this could resolve to `/usr/bin/python` or to a Python interpreter inside an `Anaconda` or `virtualenv` environment.



## Shell Tools

### Finding flags and options

You might be wondering how to find the flags and options for the commands we use. The first approach is to call the command with the `-h` or `--help` flags. `--help` or `-h` is commonly supported by many command-line programs, but it is not guaranteed in all shells or utilities. A more detailed approach is to use the `man` command, which usually provides the full manual and covers descriptions for special variables and tokens in the operating system. [TLDR pages](https://tldr.sh/) are a useful complementary resource.

### Finding files

One of the most common repetitive tasks that every programmer faces is finding files or directories. All UNIX-like systems come packaged with `find`, a powerful shell utility for locating files. `find` recursively searches for files that match specified criteria. Some examples:

```bash
# Find all directories named src
find . -type d -name src

# Find all Python files that have a folder named test in their path
find . -type f -path '*/test/*.py'

# Find all files modified in the last day
find . -mtime -1

# Find all tar.gz files with size in range 500k to 10M
find . -type f -name '*.tar.gz' -size +500k -size -10M
```

Beyond listing files, `find` can also execute commands on files that match your query. This capability can be extremely helpful in automating otherwise monotonous tasks.

```bash
# Delete all files with .tmp extension
find . -type f -name '*.tmp' -exec rm {} \;

# Find all PNG files and convert them to JPG
find . -type f -name '*.png' -exec sh -c 'convert "$1" "${1%.png}.jpg"' _ {} \;
```

Despite `find`'s ubiquity, its syntax can sometimes be tricky to remember. For instance, to find files in the current directory that match a pattern `PATTERN`, you would run `find . -name '*PATTERN*'` (or `-iname` for case-insensitive matching). You could create aliases for common use cases, but part of the shell philosophy is to explore alternative tools when appropriate.

* `fd` is a user-friendly alternative to the `find` command.
* `locate` uses a database that is updated via `updatedb`. On most systems, `updatedb` runs daily as a `cron` job.

### Finding Code

Finding files by name is useful, but often you need to search based on file *content*. A common task is to find all files that contain a specific pattern and see where in those files the pattern appears. Most Unix-like systems provide `grep`, a general-purpose tool for matching patterns in text.

`grep` has many flags that make it a versatile tool:

* `-C N` — show *N* lines of context before and after each match.
* `-v` — invert the match (print lines that do **not** match).
* `-R` — search directories recursively.

In this guide, we use ripgrep (`rg`) because it is fast and easy to use. Examples:

```bash
# Find all Python files where the requests library is imported
rg -t py 'import requests'

# Find all files (including hidden files) that do not contain a shebang line
rg -u --files-without-match "^#!"

# Find all matches of "foo" and print 5 lines after each match
rg foo -A 5

# Print match statistics (number of matched lines and files)
rg --stats PATTERN
```

### Command History

So far, we have seen how to find files. As you spend more time in the shell, you may want to retrieve specific commands you typed earlier. Pressing the up arrow shows your last command. Press it repeatedly to move backward through your shell history.

The `history` command lets you access your command history programmatically. It prints your shell history to standard output. To search it, you can pipe the output to `grep`. For example, `history | grep find` prints commands that contain the substring `find`.

In most shells, you can use `Ctrl+R` to perform a reverse search through your history. After pressing `Ctrl+R`, type a substring to search for matching commands. Press `Ctrl+R` again to cycle through additional matches.

You can enhance this workflow with `fzf`, a general-purpose fuzzy finder. It can search your command history interactively and display matches in a more convenient way.

### Directory Navigation

So far, we assumed you are already in the correct directory. How can you navigate directories more efficiently? You can define shell aliases or create symbolic links with `ln -s`, but several more advanced tools are available.

You can find frequently and recently used files and directories with:

* [`fasd`](https://github.com/clvv/fasd), which provides quick access to files and directories in POSIX shells.
* [`autojump`](https://github.com/wting/autojump), an earlier tool that inspired `fasd`.

`fasd` ranks files and directories by *frecency* (a combination of frequency and recency). By default, it adds a `z` command that lets you `cd` to a directory using a substring of a frequently and recently used path. For example, if `/home/user/files/cool_project` is used often, running `z cool` will jump to it. With `autojump`, you can achieve the same result using `j cool`.

Other tools provide a structured overview of directory contents:

* [`tree`](https://linux.die.net/man/1/tree)
* [`broot`](https://github.com/Canop/broot)
* [`ranger`](https://github.com/ranger/ranger)


## Exercises

- Read `man ls` and write an `ls` command that lists files in the following manner

  - Includes all files, including hidden files
  - Sizes are listed in human readable format
  - Files are ordered by recency
  - Output is colorized

```bash
ls -lath --color=auto
```

- Write bash functions  `marco` and `polo` that do the following. Whenever you execute `marco` the current working directory should be saved in some manner, then when you execute `polo`, no matter what directory you are in, `polo` should `cd` you back to the directory where you executed `marco`.

```bash
marco() {
    export MARCO=$(pwd)
}

polo() {
    cd "$MARCO"
}

~$ Desktop source ~/marco.sh 
~$ Desktop marco 
~$ Desktop cd ../Documents 
~$ Documents polo
~$ Desktop 
```

- Say you have a command that fails rarely. In order to debug it you need to capture its output but it can be time consuming to get a failure run. Write a bash script that runs the following script until it fails and captures its standard output and error streams to files and prints everything at the end. Bonus points if you can also report how many runs it took for the script to fail.

```bash
#!/usr/bin/env bash

count=0
while true; do
  count=$((count+1))
  ./random.sh &> out.txt
  if [[ $? -ne 0 ]]; then
    break
  fi
done

echo "found error after $count runs"
cat out.txt
```

- As we covered in the lecture `find`'s `-exec` can be very powerful for performing operations over the files we are searching for. However, what if we want to do something with all the files, like e.g. creating a zip file. As you have seen so far commands will take input from both arguments and `STDIN`. When piping commands, we are connecting `STDOUT` to `STDIN`, but some commands like `tar` take inputs from arguments. To bridge this disconnect there's the `xargs` command which will execute a command using `STDIN` as arguments. For example `ls | xargs rm` will delete the files in the current directory. Your task is to write a command that recursively finds all HTML files in the folder and makes a zip with them. Note that your command should work even if the files have spaces.

```bash
find . -type f -name "*.html" -print0 | xargs -0 zip html_files.zip
```

`xargs -0` takes the null-separated file names from `find` and passes them as arguments to the next command.

- Listing files by recency?

```bash
ls -alth
```
