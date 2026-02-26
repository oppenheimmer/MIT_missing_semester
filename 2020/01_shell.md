# Shell

## What is shell?

Computers provide a variety of I/O interfaces, such as GUIs, voice interfaces, and AR/VR. However, these interfaces are fundamentally limited in how they operate. They are preprogrammed for specific commands or tasks and are not generally composable. To fully harness a computer’s capabilities, it is preferable to send commands through a text interface, i.e., *the shell*. Shells operate using diverse, composable textual commands. They allow users to run programs, provide inputs, and view outputs. We will focus on the **B**ourne **A**gain **Sh**ell (`bash`). It is one of the most widely used shells, and its syntax is similar to other modern shells. To see a *shell prompt* (where you can type commands), you need a *terminal* application.

> A shell is a textual interface to the operating system kernel. Shell prompts (where commands are entered) are accessed through terminal applications (such as PuTTY).


## Using the shell

When you open a terminal application, you'll see a shell prompt:

```console
[missing]:~$
```

This shell prompt conveys:

- You are on the machine `missing`
- Current working directory is `~` (or the `home` directory)
- You are not the root user (via `$`).

At this prompt, you can type commands. These commands will be interpreted by the shell to execute programs or scripts. For example,

```console
~$ date
   Wed Jun  1 16:04:01 JST 2022
```
This displays the date and time of the command executed. It does not have any arguments, unlike

```console
~$ echo hello
   hello
```

We instructed the shell to execute `echo` with the argument `hello`, separated by whitespace. The `echo` program simply prints its arguments. The shell parses a command by splitting it on whitespace and then runs the program indicated by the *first* word. All subsequent words become arguments (or options). If an argument contains spaces or special characters (e.g., a directory named `"My Photos"`), you can either quote it (using `'` or `"`, e.g., `"My Photos"`) or escape the spaces (e.g., `My\ Photos`).


```console
~$ echo "Hello, world"
   Hello, world
~$ echo 'Hello World'
   Hello World
~$ echo Hello\ World
   Hello World
```

In POSIX shells (e.g., bash, dash, zsh in sh-mode), quoting controls expansion and word splitting. 

Single quotes preserve the literal value of every character inside.

   - No variable expansion
   - No command substitution
   - No arithmetic expansion
   - No pathname expansion (globbing)
   - Backslashes lose special meaning

```console
~$ name="Alice"
~$ echo '$name'
   $name
```

Double quotes preserve most characters, but still allow:

   - Variable expansion: $var
   - Command substitution: $(cmd)
   - Arithmetic expansion: $(( ))
   - Backslash escaping for certain characters
   
```console
~$ name="Alice"
~$ echo "Hello $name"
   Hello Alice
```

Read more on [quoting](https://www.gnu.org/software/bash/manual/html_node/Quoting.html) in `bash` manual. More information is in the notes section (after Exercises).


## How does the shell know your command?

Shell is a programming environment. It includes built-in commands and supports features such as variables, conditionals, loops, and functions. When the shell is asked to execute a command that does not match one of its built-in keywords, it checks an *environment variable* called `$PATH`. This variable lists the directories the shell searches for executable programs. Environment variables are initialized when the shell starts. 

> For a list of environment variables, you can run `compgen -e`.

```console
~$ echo $PATH
   /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
~$ which echo
   /bin/echo
~$ /bin/echo $PATH
   /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

When we run `echo`, the shell recognizes it as a program name and searches the colon-separated list of directories in the `$PATH` variable. When it finds the first match, it executes that program. We can determine which binary will be executed using `which`. We can also bypass `$PATH` entirely by specifying the full path to the executable, for example `/usr/bin/echo`.

> Commands are entered at the shell prompt. They execute programs (with or without arguments). The shell interprets each command by searching the directories listed in the `$PATH` variable for a matching executable.


## Navigating in shell

A `path` in the shell is the location of a directory or file, with components separated by `/` (on Linux and macOS). On POSIX systems, `/` represents the *root* of the filesystem, under which all directories and files reside. A path that starts with `/` is called an *absolute* path and specifies the complete location of a file or directory. Any path that does not start with `/` depends on the current working directory and is called a *relative* path. The current working directory can be displayed with `pwd` and changed with `cd`. To list the contents of a directory, we use the `ls` command. The `ls` command can take a path as an argument.

> Absolute paths start at the root of the filesystem (`/`) and use `/` as a separator between components. Relative paths use `.`, `..`, and `~` to navigate the directory hierarchy.

In general, when we run a program, it operates in the current working directory unless specified otherwise. For example, it typically searches for input files in the current directory and creates output files there. To ensure that a program (or executable) is found and executed, we either add its directory to `$PATH` or specify its absolute (or relative) path explicitly.

```console
~$ ls

~$ cd ..
/home$ ls
       missing

/home$ cd ..
/$ ls
   Applications Users    cores   home      sbin  var
   Library      Volumes  dev     opt       tmp
   System       bin      etc     private   usr
```

Notes:

- `~` refers to the home directory, and running `cd` without arguments takes you there regardless of your current location. The tilde can also be used when specifying paths, for example `~/missing/classes/`.
- A dash (`-`) used with `cd` (i.e., `cd -`) returns you to the previous directory. It is a convenient way to toggle between the current and last visited directories.
- Unless a path is provided as its first argument, `ls` lists the contents of the current directory.
- Many commands accept flags and options that modify their behavior. Typically, running a program with the `-h` or `--help` flag displays usage information (e.g., `ls --help`). More detailed documentation is usually available in the manual pages, accessed with `man`.

> In shell commands, a keyword that does not accept a value and has a boolean value is called a `flag`. If it accepts one or more values or arguments, it is an `option`. Flags are typically prefixed with `-`, whereas options often use `--`. An option that does not take an argument is often called a flag because technically, both are options.


## Access control

When we use `ls -l` or `ls -al`,

```console
~$ ls -l /home
   drwxr-xr-x 1 sourav users  4096  Feb 8 2022  missing
```

Such a command shows detailed information about each file or directory in the current working directory.  

First, a `d` at the beginning of the line indicates that the entry is a directory. Conversely, if it is absent, the entry is a file. Then follow three groups of three characters each. These indicate which permission bits are set, in the following order: owner (`sourav`), group (`users`), and others (`others`). A `-` indicates that the given principal does not have the corresponding permission.

For files,

- `r` -> Read permission. File can be read, if set
- `w` -> Write permission. File can be written or emptied, if set
- `x` -> Execute permission. File can be executable if set

For directories,

- `r` -> scan permission. Principal can see inside the directory i.e. list directory contents.
- `w` -> edit permission. Moving, renaming, and deletion in the directory i.e. changes to directory
- `x` -> search permission. Principal allowed to enter the directory i.e `cd` into directory


Some other handy programs to know at this point are:

- `mv` to rename or move a file; takes two paths as arguments
- `cp` to copy a file; takes source and destination arguments
- `rm` remove a file. `-r` makes it recursively delete everything
- `mkdir` to make a new directory
- `rmdir` to remove empty directory


## Connecting programs

We may want to chain different programs together to make their utility more powerful. In the shell, programs have two primary control *streams*: the `input stream` and the `output stream`. When a program needs input, it reads from the input stream; when it produces output, it writes to the output stream. We can also rewire these streams when we need to perform more complex tasks. The simplest form of redirection uses angle brackets: `< file` and `> file`. These allow you to redirect a program’s input and output streams to files, respectively:

```console
~$ echo hello > hello.txt
~$ cat hello.txt
   hello

~$ cat < hello.txt
   hello

~$ cat < hello.txt > hello2.txt
~$ cat hello2.txt
   hello
```

`cat` is a program that concatenates file contents to the output stream. When given file names as arguments, it prints the contents of the files in sequence. (When `cat` is not given any arguments, it prints content from the standard input stream to the standard output stream indefinitely.) We can use `>>` to append to a file.

```console
~$ cat hello.txt hello2.txt
   hello
   hello
```

Input-output redirection really shines in the use of *pipes*. The `|` operator lets you chain programs such that the output of the first is the input to the next. These programs do not know about each other's I/O. They are connected by a *pipe*.

```console
~$ ls -l / | tail -n1
   drwxr-xr-x 1 root  root  4096 Jun 20  2019 var
```

## Super user

On most Unix-like systems, one user is special: `root`, with user ID 0. The root user is above (almost) all access restrictions and can create, read, edit, and delete any file on the system. You will not usually log into your system as the root user, though, since it is too easy to accidentally break something. Instead, you will use the `sudo` command—short for *superuser do*. When you receive a “permission denied” error, it is usually because you need to perform the action as root with elevated privileges.

To illustrate its power, `sudo` can even be used to edit system parameters. You need superuser privileges in order to write to the `sysfs` filesystem mounted under `/sys`. `sysfs` exposes a number of kernel parameters as files. One can easily reconfigure the kernel on the fly without specialized tools. For example, the brightness of your laptop’s screen is exposed through a file called `brightness` (on ThinkPads):

```console
/sys/class/backlight
```

By writing a value into that file, we can change the screen brightness. Perhaps, the first instinct might be to do something like:

```console
$ sudo find -L /sys/class/backlight -maxdepth 2 -name '*bright*'
  /sys/class/backlight/thinkpad_screen/brightness

$ cd /sys/class/backlight/thinkpad_screen
$ cat max_brightness
  1060
$ sudo echo 3 > brightness
  An error occurred while redirecting file 'brightness' open: Permission denied
```

This error may come as a surprise. After all, we ran the command with `sudo`. The key takeaway is:

> Pipe and redirection operations such as `|`, `>` and `<` are handled by the shell, not by the program itself. `sudo` only applies to the executable it precedes.

`echo` does not know anything about `|`. It simply reads from its standard input and writes to its standard output, whatever those may be. In the case above, the shell (which is authenticated as a normal user) attempts to open the brightness file for writing before assigning it as the output of `sudo echo`. This operation fails because the shell is not running as root and therefore does not have permission to write to the file. Only the `echo` command is executed with superuser privileges.

We can change the shell to `root` to bypass this:

```console
$ su -
  [sudo password]: *****

# echo 3 > brightness
```

Using our understanding of shell, we can also work around this by:

```console
~$ echo 1060 | sudo tee brightness
```

The `tee` program writes to a file and also outputs the content to standard output, similar to `echo`. Since `tee` is the process that opens `/sys/brightness` for writing, and it is running as `root`, the permissions are correctly applied and the write operation succeeds.



**Exercise:**

- Use touch to create a new file called `missing` in `/tmp/semester`.

```console
~$ touch /tmp/semester/missing
```

- Write the following one line at a time:

```console
#!/bin/sh

curl --head --silent https://missing.csail.mit.edu
```

This is how to do it,

```console
~$ echo '#!/bin/sh' > missing
~$ echo 'curl --head --silent https://missing.csail.mit.edu' >> missing
```

- Try executing the file:

> It will not work since 'x' bit is missing. It can be fixed by `chmod +x`. You can play around with other bits e.g. `chmod -w` to remove write permission.

- Does `sh missing` work? Why?

> Yes it works. Though `./missing` will not work by its own currently, it can be set to execute by `chmod +x missing`. If you make your file executable and run it with `./file`, then the kernel will see that the first two bytes are `#!`, which means it's a script-file. The kernel will then use the rest of the line as the interpreter, and pass the filename as first argument. So, it runs `/bin/bash [filename]`. Under the hood it is `execve("/bin/sh", ["/bin/sh", "missing"], ...)`

> For a shell to execute the script, it only needs to be able to read the file i.e. `r` bit is important. If it is a valid script it can be run by passing as argument to `sh`/`bash`/`zsh`. To really prohibit execution, take away the read bit. When you are running `sh file.sh`, you are executing `sh [arg]` not `./file`.

- Use pipes to write the “last modified” date

```console
~$ ./missing | grep "last" > last.txt
```

## Notes on Quoting

There are three quoting mechanisms: the escape character, single quotes, and double quotes:

- **Escape**: A non-quoted backslash \ is the Bash escape character. It preserves the literal value of the next character that follows, with the exception of newline.

- **Single Quotes**:
Enclosing characters in single quotes preserves the literal value of each character within the quotes. A single quote may not occur between single quotes

- **Double quotes** Enclosing characters in double quotes preserves the literal value of all characters within the quotes, with the exception of `$`,`` ` ``, `\` and, when history expansion is enabled, the character ``!``. The characters ``$`` and `` ` `` retain their special meaning within double quotes. The parameters `*` and `@` have special meanings.

```console
~$ echo Hello\ OK
   Hello OK

~$ echo Hello\\ OK
   Hello\ OK

~$ echo Hello\\nOK
   Hello
   OK

~$ echo 'Hello\\r OK'
   Hello\r OK

~$ echo "Hello\\rOK"
   OKllo

~$ echo Hello\\n\\rOK
   Hello
   OK
```
