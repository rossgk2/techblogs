# Background

The first successful mass-market personal computer was the Apple II, and was released in 1975. It would be nine years until the first computer to provide the user a graphical user interface, the Apple Macintosh, was released in 1984. (And it was four more years until Microsoft released its first widely successful GUI-based OS, Windows 3.0.)

Before the time of graphical user interfaces, computer users would write and execute text commands in what is known as a *command line interface*, *terminal*, or *shell*.

The command line is still alive and well today (for programmers, at least) because basic command line knowledge is necessary to compile and test programs in essential programming languages like C, C++, C#, Java, and Python\*.

This article gives an overview of the shells in use today.

\* Yes, it's true that integrated development environments (IDEs) abstract command line details away and thus make it possible to compile and run programs by clicking user-friendly icons- but in order to interact with the IDE settings in an informed way, you have to know what's going on behind the scenes! 

# Shells by OS

Modern shells can be roughly grouped into three categories:

* Unix shells
* Windows shells that are pre-PowerShell
* Windows PowerShell

Shells within the same category use similar syntax, and shells in different categories use differing syntax.

## Unix (MacOS/Linux)

| Shell name                  | Notes                                        | Implemented with | File extension of shell scripts |
| --------------------------- | -------------------------------------------- | ---------------- | ------------------------------- |
| sh ("Bourne shell")         | Written by Stephen Bourne at Bell Labs       | C                | No extension or .sh             |
| bash ("Bourne again shell") | Successor to sh and most popular shell today | Mostly C         | No extension, .sh, or .bash     |
| zsh ("Z shell")             | Default shell in newer MacOS versions        | C                | No extension or .zsh            |

## Windows CMD

| Shell name | Notes                                                        | Implemented with | File extension of shell scripts                            |
| ---------- | ------------------------------------------------------------ | ---------------- | ---------------------------------------------------------- |
| CMD        | Default Windows shell having some backwards compatibility with MS-DOS, which as the interface provided by Windows in the 1980s. | C                | .bat ("batch file") or .cmd (essentially the same as .bat) |

## Windows PowerShell

| Shell name | Notes                                                        | Implemented with                      | File extension of shell scripts |
| ---------- | ------------------------------------------------------------ | ------------------------------------- | ------------------------------- |
| PowerShell | Though it has some backwards compatibility with CMD, its built-in commands use conventions that differ from the conventions used by CMD's build-in commands. | Mostly C#, some C and C++, some other | .ps1                            |

# Structure of a shell command

All shell commands generally have ways of accepting the following kinds of input:

* named arguments that accept input
* named arguments that don't accept input ("flags")
* unnamed arguments ("positional arguments")

We can see these concepts at play in the following bash command:

```
curl --request "GET" --include "https://google.com/"
```

The `curl` command is the "client for URLs" command. It sends a request of the specified type to the specified URL. In this command,

* `request` is a named argument, and its value is "GET".
  * Specifying that `request` is "GET" tells `curl` that the type of HTTP request it is to send to the URL is a "GET" request, and not a "POST" or any other type of request.
* `include` is a named argument that doesn't accept input, i.e., it is a flag.
  * When `include` is present, `curl` prints the HTTP response it receives back from the URL.
  * Specifying `include` is somewhat like saying that `include` is `true`. If we didn't specify `include`, and executed `curl --request "GET" "https://google.com"`, then we would have effectively set `include` to `false`.
* `"https://google.com"` is the value of an unnamed argument.
  * `curl` interprets the first unnamed argument to be a URL and attempts to visit that URL.

## Inside a shell command

Every shell comes with a set of standard commands it can run. Most bash commands, including the standard ones such as `curl`, are written in C. The C code underlying `curl` looks something like this:

```c
/* curl.c */
int main(int argc, char* argv[])
{
	/* ... */
}
```

(Note: `char* argv[]` denotes an array of `char*` values named `argv`. A  `char*` is by definition a pointer to a `char` value. And a pointer to a 	`char` value is essentially a string. Thus `char* argv[]` denotes an array of strings.)

When a user executes the code

```bash
curl --request "GET" --include "https://google.com/"
```

the bash shell passes the result of splitting the command text by spaces-not-enclosed-by-quotes as the value for `argv` and the length of this array as `argc`. So, we would have

```
argv[0] == "curl"
argv[1] == "--request"
argv[2] == "GET"
argv[3] == "https://google.com/"
argc == 4
```

How the program interprets and makes use of the input passed to it (i.e. how it makes use of `argv`) is for the program author to decide. We know from above how `curl` is expected to behave, but any number of crazy things could be done.

## Parsing conventions

As you would expect, there are popular conventions for how programs parse their input. The intuitive Unix style is the most popular. It continues to grow more popular: CMD and PowerShell have increasingly added more and more support for Unix-like syntax.

### Unix style (most popular)

The essentials of a Unix-style shell command are as follows:

* `--` is used to denote named arguments; both named arguments that accept input, and named arguments that don't accept input ("flags")
  *  e.g. `ls --time-style "long-iso"`
  * e.g.  `ls --directory` 

* `-` is used as a single-letter short-form for named arguments
  * e.g. `ls -d` is the same as `ls --directory`.
* Strings following a single dash `-` are interpreted to be the combination of short-form named arguments
  * e.g. `ls -da` is the same as `ls -d -a`, which is the same as `ls --directory --all`

* Arguments can be assigned values via space separation (e.g. `program --arg "value"` and `program -a "value"`) or with an `=` sign (e.g. `program --arg="value"` and `program -a="value"`)

* Positional arguments are typically required to either be all before or all after the named arguments[^1]. 
  * Some commands require the user to denote the end of all named arguments with the string ` -- `.
* Some Unix commands support "sub-options" that are only available when another argument takes on a particular value.
  * This is in fact the principle behind "subcommands"[^2].


[^1]: It is technically possible for a command to successfully parse unnamed arguments that are "mixed in" with named arguments, but allowing this makes commands unreadable. So, most commands enforce that all unnamed arguments either go before all named arguments or after all named arguments.

[^2]: A positional argument `parg` is said to be a *subcommand* if (1) it is the first positional argument, and the form of the remaining command (the portion of the command not including `parg`) depends on `parg` or (2) the preceding positional argument is a subcommand, and the form of the remaining command (the portion of the command not including the preceding positional arguments nor `parg` ) depends on `parg`.

### CMD and PowerShell

* In CMD, `/` is used to denote named arguments.
* In PowerShell, `-` is used to denote named arguments.
* Some CMD and PowerShell commands support both long-form named arguments and short-form named arguments. Combining short-form named arguments in the Unix style is not supported for such commands[^3].
* Other CMD and PowerShell commands support short-form named arguments and also Unix-style combination of short-form named arguments, but not long-form named arguments[^3].

* CMD commands increasingly support Unix-style use of `--` and `-`.

[^3]: If short-form arguments can be combined as in the Unix style, then using the same symbol for long-form and short-form named arguments is ambiguous. One can have either (1) Unix-style short form argument combination or (2) use the same symbol for long-form and short-form named arguments, but it's impossible to have both without introducing ambiguity.

### Real world inconsistency

Remember that any command in any shell can define any syntax it wants. It is very possible to encounter commands on a Unix system that conform to non-Unix standards. For example, if Java is installed, then the `java` executable uses `-` for named arguments in a PowerShell sort of style.  

## Environment variables and `PATH`

* An *environment variable* is a variable that is known to the entire OS.
* Environment variable identifiers are case-sensitive on Unix and case-insensitive on Windows.
* In both Unix and Windows, the value of the `PATH` environment variable is a string that consists of paths to files separated by a delimiter. On Unix, the delimiter is the colon :. On Windows, the delimiter is the semicolon ;.
* Normally, to execute a program in a shell, one has to specify the full path to that program. For example, one would run something like  `C:\Program Files\Java\jdk-1.8.0\bin\javac` to run the Java 1.8 compiler, and something like `C:\Program Files\Java\jdk-1.8.0\bin\javac` to run the Java 1.8 executable.
* Executables whose paths appear in `PATH` do not need to be specified explicitly like this, though, as the shell will search all paths listed in PATH before it attempts to execute any command. So, if we add `C:\Program Files\Java\jdk-1.8.0\bin\`  to `PATH`, then we can simply run `javac` to run the compiler and `java` to run the executable.
* Windows includes the current directory `.` in `PATH`. Unix does not.
  * From a security perspective, the Unix approach of not including `.` in PATH is best because, if someone tricked you into putting a malicious executable called `ls` in your current directory, and then had you run `ls`, you'd run their malicious `ls` instead of the system `ls`.
* In Unix shells, have to run programs with code like `./program`. Simply `program` works in CMD.

# Tutorial: navigating file systems with bash

The following is a quick tutorial for navigating the file system of your computer with a bash shell. 

First, run the following command. 

```
pwd
```

It is the "print working directory" command, and therefore prints out your working directory as text to the screen. Your *working directory* is the folder that the shell is currently inspecting; it is analogous to the folder you currently have open in a file explorer. Now, execute

```
ls
```

This "list" command prints out all of the files in the working directory. If you wanted to only see folders in the current working directory, you could execute 

```
ls --directory */
```

The value `*/`  , which is passed in as an unnamed argument, causes the `ls` command to only search for files whose path is of the form `*/`, i.e., files whose paths end in a slash /. The `--directory` flag causes the `ls` command

```
cd
```

## bash on Windows

* Windows Subsystem for Linux
* Git Bash
