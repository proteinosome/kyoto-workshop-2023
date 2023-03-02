Command Line Crash Course
================
Khi Pin, Chua
19/01/2022

- [Command Line Crash Course](#command-line-crash-course)
- [Background and Introduction](#background-and-introduction)
  - [Why command line?](#why-command-line)
  - [Cloud compute instances](#cloud-compute-instances)
- [Simple and useful Bash commands](#simple-and-useful-bash-commands)
  - [`pwd` (Present working directory)](#pwd-present-working-directory)
  - [Useful to know: Unix folder structures](#useful-to-know-unix-folder-structures)
  - [`cd` (Change directory)](#cd-change-directory)
  - [`ls`](#ls)
  - [How to create a folder? `mkdir` “make a directory”!](#how-to-create-a-folder-mkdir-make-a-directory)
  - [`rm` and `rmdir`: Removing files or directories (Warning, be careful!)](#rm-and-rmdir-removing-files-or-directories-warning-be-careful)
  - [`touch` to create empty file](#touch-to-create-empty-file)
  - [`cp` to copy files/directories around](#cp-to-copy-filesdirectories-around)
  - [`mv` to move files/directories around](#mv-to-move-filesdirectories-around)
  - [View any text file on command line with `cat`, `less`, `head` and `tail`](#view-any-text-file-on-command-line-with-cat-less-head-and-tail)
  - [`history` of commands](#history-of-commands)
  - [`grep`-ing at the needle in the haystack](#grep-ing-at-the-needle-in-the-haystack)
  - [`cut` the specific columns you want to see](#cut-the-specific-columns-you-want-to-see)
  - [`sort` the way you like it](#sort-the-way-you-like-it)
- [Some useful command line’s “magic”](#some-useful-command-lines-magic)
  - [The “wildcard” magic](#the-wildcard-magic)
  - [The “Tab” magic](#the-tab-magic)
  - [The “pipe” magic](#the-pipe-magic)
  - [The “redirect” magic](#the-redirect-magic)
  - [Other useful tools and topics not covered](#other-useful-tools-and-topics-not-covered)

# Background and Introduction

## Why command line?

-   Researchers often exchange data (Sequencing data, clinical data,
    etc) in file format that’s not easily manipulated with graphical
    user interface (GUI).
    -   The data may be too big and the softwares are not designed to
        work with them.
    -   Some softwares may accidentally introduce formatting errors.
        E.g. numbers get changed to date.
-   With command line, it’s easy to peek quickly into what do you have
    on hand.
-   Unix tools are ubiquitous and fast.
-   Many tasks can be simplified and automated, saving valuable time.
    There are tasks that can be done quickly just using command line
    without going into a full-blown programming language.
-   Combining scripting and command line softwares is key to
    reproducible research.
-   Finally, access to high performance computing (HPC) resources is
    often done via command line. Today’s approach of using SSH is just
    one example.

## Cloud compute instances

-   After logging into a server/computer via SSH in a terminal, you will
    often see a “prompt” that contains informations on the user and the
    name/address of the computer. The “prompt” ends in `$`. You can
    start typing any command after the `$` mark and press `Enter` to
    execute the command. For example, try typing `echo "hello world` and
    press `Enter` to show a message “hello world”

``` bash
echo "hello world"
```

    ## hello world

-   The command line “language” that you are using is called `Bash`
    which is the most common type of command line language you will see
    on most Unix-based system. Other shell/command language includes
    `Zsh` which is the default language for many newer MacOS system.
-   Most of the tools we’ll use can be found universally in most
    Unix-based systems regardless of the shell language.
-   **Importantly**, you can look at the manual for any command simply
    by typing `man COMMAND`. For example, try `man echo` to read the
    manual of `echo` (You can press `q` to exit from the manual).
-   `man` provides very detailed documentations for many tools. However,
    most of them also come with a short “help” that you can print on the
    command line for quick reference in the form of “`command --help`”.
    For example, try `grep --help`
-   `--help` is almost always available for many third-party command
    line softwares, including PacBio’s.

# Simple and useful Bash commands

## `pwd` (Present working directory)

-   The `pwd` command simply tell you where are you currently.

``` bash
pwd
```

    ## /Users/khipinchua/OneDrive/Documents/PacBio/trainings_meetings/2022-1-19_KDRI_Japan_IsoSeq_Workshop/command_line_crash_course

## Useful to know: Unix folder structures

-   Whenever you see something like `/home/users/file1.txt`, it is a
    location on the server where the file is located in.
-   File paths always start with “/” to indicate the “root” (uppermost
    directory containing the file). Imagine peeling an onion layer by
    layer before you get to the innermost layer which is the file that
    you want. The outermost layer/skin is the “root”
    -   Exception, you may see something like `~/file1.txt`. The `~`
        sign is an alias that represents the “home folder” for the user
        (You). In most Linux, your home folder is usually
        `/home/users/USERNAME`. In other words, if your username is
        `kpin`, your home folder would be `/home/users/kpin` and typing
        `~` is the same as typing `/home/users/kpin`
    -   For example, you can type `echo ~` to see what `~` means.

``` bash
echo ~
```

    ## /Users/khipinchua

## `cd` (Change directory)

-   The `cd` command means “change directory”. Simply put, it helps you
    to bring your “present working directory” to another place! Try:

``` bash
pwd
```

    ## /Users/khipinchua/OneDrive/Documents/PacBio/trainings_meetings/2022-1-19_KDRI_Japan_IsoSeq_Workshop/command_line_crash_course

``` bash
# We will talk about "~" later
cd ~/command_line_crash_course/
pwd
```

    ## /Users/khipinchua/command_line_crash_course

-   You can go back to the folder one layer up (e.g. before you enter
    the `cd` command) by typing:

``` bash
cd ..
pwd
```

    ## /Users/khipinchua/OneDrive/Documents/PacBio/trainings_meetings/2022-1-19_KDRI_Japan_IsoSeq_Workshop

-   In general, `..` refers to the directory one “layer” up, and `.`
    refers to the current directory.

-   **Question**: What would `cd ~` does?

-   **Question**: What would `cd ../../` do?

## `ls`

-   `ls` can be used to list the files and folders in the current
    directory

``` bash
# Let's go into the crash course folder
cd ~/command_line_crash_course/
ls
```

    ## command_line_crash_course.Rmd
    ## command_line_crash_course.md
    ## folder
    ## folder_x
    ## folder_y
    ## large_text.txt
    ## small_text.tsv

-   `ls` has parameters that allows you to format the output according
    to your liking, e.g.:
    -   `-l` output the files line by line and show additional
        informations such as permissions
    -   `-h` output the sizes in human-readable format (e.g. in
        kilobytes instead of bytes)
    -   `-t` sort the files by date of which the file was modified
    -   `-a` shows hidden files
    -   `-R` list “recursively”, i.e. list all the files/directories as
        well as those in the sub-directories
    -   `-d` list just directories
-   The parameters can be combined instead of typing `-` multiple times,
    e.g. `-lhtar`

``` bash
ls -lhtaR
```

    ## total 416
    ## -rw-r--r--   1 khipinchua  staff    17K Jan 18 19:16 command_line_crash_course.Rmd
    ## -rw-r--r--   1 khipinchua  staff    14K Jan 18 18:25 .Rhistory
    ## drwxr-xr-x  10 khipinchua  staff   320B Jan 18 18:25 .
    ## -rw-r--r--   1 khipinchua  staff    24K Jan 18 18:06 command_line_crash_course.md
    ## drwxr-xr-x  19 khipinchua  staff   608B Jan 17 16:52 ..
    ## -rw-r--r--   1 khipinchua  staff   153B Jan 10 21:23 small_text.tsv
    ## drwxr-xr-x   2 khipinchua  staff    64B Jan 10 21:07 folder_y
    ## drwxr-xr-x   2 khipinchua  staff    64B Jan 10 21:07 folder_x
    ## -rw-r--r--   1 khipinchua  staff   139K Jan 10 20:55 large_text.txt
    ## drwxr-xr-x   4 khipinchua  staff   128B Jan 10 18:33 folder
    ## 
    ## ./folder_y:
    ## total 0
    ## drwxr-xr-x  10 khipinchua  staff   320B Jan 18 18:25 ..
    ## drwxr-xr-x   2 khipinchua  staff    64B Jan 10 21:07 .
    ## 
    ## ./folder_x:
    ## total 0
    ## drwxr-xr-x  10 khipinchua  staff   320B Jan 18 18:25 ..
    ## drwxr-xr-x   2 khipinchua  staff    64B Jan 10 21:07 .
    ## 
    ## ./folder:
    ## total 8
    ## drwxr-xr-x  10 khipinchua  staff   320B Jan 18 18:25 ..
    ## -rw-r--r--   1 khipinchua  staff    20B Jan 10 18:33 file1.txt
    ## drwxr-xr-x   4 khipinchua  staff   128B Jan 10 18:33 .
    ## -rw-r--r--   1 khipinchua  staff     0B Jan 10 18:29 file2.txt

-   Sometimes you executed a command that’s taking too long or it was
    being executed accidentally, you can interrupt the command while
    it’s running by pressing `Ctrl + C` (`Ctrl` and `c` key at the same
    time).
    -   Note that this does **not** reverse what has been done by the
        command before you interrupt it, e.g. if you accidentally
        execute a command to delete a file, pressing `Ctrl + C` will not
        recover it
-   Run the following command line which literally tells the command
    line to wait for 5000 seconds and try to terminate it.

``` bash
sleep 5000
```

-   You can add a folder path to `ls` command to directly list the files
    and folders inside that path instead of listing what’s in the
    present working directory.

``` bash
ls -lh folder
```

    ## total 8
    ## -rw-r--r--  1 khipinchua  staff    20B Jan 10 18:33 file1.txt
    ## -rw-r--r--  1 khipinchua  staff     0B Jan 10 18:29 file2.txt

## How to create a folder? `mkdir` “make a directory”!

-   The command `mkdir` makes a directory at your present working
    directory.
    -   **Question** How do you find your present working directory?

``` bash
# Remember "-d" list only directories. If "tmpdir" does not exist, it'll
# complain
ls -ld tmpdir
```

    ## ls: tmpdir: No such file or directory

``` bash
mkdir tmpdir
```

``` bash
ls -ld tmpdir
```

    ## drwxr-xr-x  2 khipinchua  staff  64 Jan 18 19:20 tmpdir

-   Sometimes you want to make a folder 2 “levels” down, for example a
    folder called `test2` inside a folder `test1`, you can either:

``` bash
mkdir test1
mkdir test1/test2
```

``` bash
# This will list everything inside "test1" folder
ls -lh test1
```

    ## total 0
    ## drwxr-xr-x  2 khipinchua  staff    64B Jan 18 19:20 test2

or you can use the `-p` parameter to instruct `mkdir` to create any
“parents” folder necessary for what you want to create:

``` bash
mkdir -p test2/test3
```

``` bash
# This will list everything inside "test1" folder
ls -lh test2
```

    ## total 0
    ## drwxr-xr-x  2 khipinchua  staff    64B Jan 18 19:20 test3

-   **Question**: Can you make a directory called `isoseq_cli` inside
    the folder `workshop_data`?
-   **Question**: What would `mkdir ./test1` do?

## `rm` and `rmdir`: Removing files or directories (Warning, be careful!)

-   `rm FILE` is used to delete a file. For example, if we want to
    delete a file called “tmp1.txt”:

``` bash
# There's a file called "tmp1.txt" here
ls tmp1.txt
```

    ## tmp1.txt

``` bash
# Let's delete it
rm tmp1.txt
# Is it still there?
ls tmp1.txt
```

    ## ls: tmp1.txt: No such file or directory

-   What if you want to delete a folder/directory? Both `rmdir` or
    `rm -r` (remove recursively) can be used:

``` bash
# We want to delete the folders created just now "tmpdir", "test1" and "test2"
ls -ld tmpdir test1 test2
```

    ## drwxr-xr-x  3 khipinchua  staff  96 Jan 18 19:20 test1
    ## drwxr-xr-x  3 khipinchua  staff  96 Jan 18 19:20 test2
    ## drwxr-xr-x  2 khipinchua  staff  64 Jan 18 19:20 tmpdir

``` bash
rm -r tmpdir test1 test2
ls -ld tmpdir test1 test2
```

    ## ls: test1: No such file or directory
    ## ls: test2: No such file or directory
    ## ls: tmpdir: No such file or directory

``` bash
# Make the folder again so we can delete it with rmdir
mkdir tmp_folder
ls -ld tmp_folder
```

    ## drwxr-xr-x  2 khipinchua  staff  64 Jan 18 19:20 tmp_folder

``` bash
# With rmdir
rmdir tmp_folder
# Still there?
ls -ld tmp_folder
```

    ## ls: tmp_folder: No such file or directory

-   One crucial difference is that `rmdir` will not remove a folder with
    any content inside (including empty folder(s)), so it’s “safer” than
    `rm -r` which \*\*will\* remove a folder even if the folder is not
    empty.

-   **Question**: Can you make a directory called `isoseq` inside a
    folder called `japan`, then delete them?

## `touch` to create empty file

-   There’s a utility called `touch` that creates empty file. Says for
    example you want to create an empty text file to practice the `rm`
    command:

``` bash
# Look for a file called "useless.txt"
ls -lht useless.txt
```

    ## ls: useless.txt: No such file or directory

``` bash
# Create an empty file called "useless.txt"
touch useless.txt
ls -lht useless.txt
```

    ## -rw-r--r--  1 khipinchua  staff     0B Jan 18 19:20 useless.txt

``` bash
# Delete it
rm useless.txt
ls -lht useless.txt
```

    ## ls: useless.txt: No such file or directory

-   Note that if the file that you are already `touch`-ing already
    exists, `touch` will only update the timestamp (modified time of the
    time) without creating or modifying the content of the file.

## `cp` to copy files/directories around

-   `cp FILE/FOLDER DESTINATION` is used to make a copy of the file and
    folder that you point it to.
-   For example, let’s create an empty file called “file1.txt” and make
    a copy of it called “file2.txt”

``` bash
# Look for a file called "file1.txt"
ls -lht file1.txt
```

    ## ls: file1.txt: No such file or directory

``` bash
# Create an empty file called "file1.txt"
touch file1.txt
ls -lht file1.txt
```

    ## -rw-r--r--  1 khipinchua  staff     0B Jan 18 19:20 file1.txt

``` bash
# Make a copy of it called file2.txt
cp file1.txt file2.txt
ls -lht file1.txt file2.txt
```

    ## -rw-r--r--  1 khipinchua  staff     0B Jan 18 19:20 file2.txt
    ## -rw-r--r--  1 khipinchua  staff     0B Jan 18 19:20 file1.txt

``` bash
# What happens if you copy the file to itself?
cp file1.txt file1.txt
```

    ## cp: file1.txt and file1.txt are identical (not copied).

``` bash
# You can also copy the file into a folder
mkdir tmp_folder
# Empty inside
ls -lhtr tmp_folder
```

    ## total 0

``` bash
cp file1.txt tmp_folder/
# file1.txt has a copy inside tmp_folder now
ls -lht tmp_folder
```

    ## total 0
    ## -rw-r--r--  1 khipinchua  staff     0B Jan 18 19:20 file1.txt

``` bash
# Let's delete unused files and folders
rm -r file1.txt file2.txt tmp_folder
```

## `mv` to move files/directories around

-   `mv FILE/FOLDER DESTINATION` moves the file/folder to a destination.

``` bash
# Make a directory called tmp_folder and a file called file1.txt
mkdir tmp_folder
touch file1.txt
ls -lh file1.txt tmp_folder
```

    ## -rw-r--r--  1 khipinchua  staff     0B Jan 18 19:20 file1.txt
    ## 
    ## tmp_folder:
    ## total 0

``` bash
# Now let's move file1.txt into tmp_folder
mv file1.txt tmp_folder/
# file1.txt is now in the folder tmp_folder
ls -lht file1.txt tmp_folder
```

    ## ls: file1.txt: No such file or directory
    ## tmp_folder:
    ## total 0
    ## -rw-r--r--  1 khipinchua  staff     0B Jan 18 19:20 file1.txt

-   `mv` is also used to rename file/folder, since renaming is identical
    to moving a file with “name1” to a file with “name2”!

``` bash
mv tmp_folder tmp_folder_renamed
# The "-d" parameter list only the folder without listing what's in the folder
ls -ld tmp_folder
```

    ## ls: tmp_folder: No such file or directory

``` bash
ls -ld tmp_folder_renamed
```

    ## drwxr-xr-x  3 khipinchua  staff  96 Jan 18 19:20 tmp_folder_renamed

``` bash
# Delete the folder after use
rm -r tmp_folder_renamed
```

-   **Question**: What if we want to make a copy of the **folder**?
    (Hint: Copy recursively)
-   Notice that for both copy and moving into a folder, there’s a
    trailing “`/`” following the destination folder?
    (e.g. `cp file1.txt tmp_folder/` and `mv file1.txt tmp_folder/`) If
    the destination folder does not exist, the `cp` and `mv` command
    will either make a copy (with `cp`) or rename (with `mv`) your
    file/folder with the supposedly folder name.

## View any text file on command line with `cat`, `less`, `head` and `tail`

-   We often want to “peek” at the content of a text file (e.g. a
    sequence FASTA, although with PacBio the sequence may be too long to
    fit into the screen!). This can be done via `cat`:

``` bash
# There's thousands of lines printing on the screen!
cat large_text.txt
```

-   As you can see, sometimes the text is too large! You can use the
    “pager” utility called `less` that opens up the text file but allow
    you to navigate “page by page”. You can move up and down using the
    arrow key, or use `w` and `Spacebar` to move up and down by one
    page. Press `q` or `Ctrl + C` to exit from `less`.

``` bash
less large_text.txt
```

-   What if you want to look at just the first few lines or the last few
    lines? Two intuitively named tools called `head` and `tail` come to
    rescue:

``` bash
# First few lines
head large_text.txt
```

``` bash
# Specify the number of lines you want to see with "-"
head -5 large_text.txt
```

    ## japan_1  iso-seq line 1
    ## japan_2  iso-seq line 2
    ## japan_3  iso-seq line 3
    ## japan_4  iso-seq line 4
    ## japan_5  iso-seq line 5

``` bash
# Tail few lines
tail large_text.txt
```

``` bash
# Similarly, specify the number of line with "-"
tail -5 large_text.txt
```

    ## japan_4996   iso-seq line 4996
    ## japan_4997   iso-seq line 4997
    ## japan_4998   iso-seq line 4998
    ## japan_4999   iso-seq line 4999
    ## japan_5000   iso-seq line 5000

## `history` of commands

-   You can use the arrow key to go back and forth to the previous
    commands (and all other previous commands) that you’ve executed.

-   Alternatively, in most shells/terminals you can press `Ctrl + R` on
    your keyboard, and search for the command using specific keywords
    (e.g. try typing `ls` after pressing `Ctrl + R`).

-   Most of the shells come with a command called `history` that will
    show you a history of the command that you’ve typed.

-   **Try**: Combine `history` with `tail` to look at the last few
    commands you’ve executed.

``` bash
history | tail -5
```

## `grep`-ing at the needle in the haystack

-   `grep` is a very powerful command that uses “Regular expression” to
    search for texts in a file.
-   In the most basic use, `grep` searches for the regular expression
    pattern provided and output the lines that matches it
-   Let’s say we want to find “line 1254” in the `large_text.txt` file:

``` bash
# Search for line 1254
grep 'line 1254' large_text.txt
```

    ## japan_1254   iso-seq line 1254

-   Regular expression is very powerful tool. Search for “regular
    expression tutorial” on the internet if you want to learn more about
    it. For example, if I want to find any line that ends with “5”

``` bash
# The $ "anchor" looks for anything that ends with the character before it
# Note that the "-m" parameter here tells grep to stop searching after finding
# 10 matches
grep -m 10 '5$' large_text.txt
```

    ## japan_5  iso-seq line 5
    ## japan_15 iso-seq line 15
    ## japan_25 iso-seq line 25
    ## japan_35 iso-seq line 35
    ## japan_45 iso-seq line 45
    ## japan_55 iso-seq line 55
    ## japan_65 iso-seq line 65
    ## japan_75 iso-seq line 75
    ## japan_85 iso-seq line 85
    ## japan_95 iso-seq line 95

## `cut` the specific columns you want to see

-   In bioinformatics, we often work with `csv` and `tsv` files, which
    are files whereby a table is saved in a format of which the columns
    are separated by comma (`csv`) or tab (`tsv`). Sometimes, we only
    need a specific column:

``` bash
# What's inside small_text.tsv?
cat small_text.tsv
```

    ## japan_1  iso-seq line 1  1
    ## japan_2  iso-seq line 2  2
    ## japan_3  iso-seq line 3  3
    ## japan_4  iso-seq line 4  4
    ## japan_5  iso-seq line 5  5
    ## japan_11 iso-seq line 11 11

``` bash
# Only need the first column of the file
cut -f1 small_text.tsv
```

    ## japan_1
    ## japan_2
    ## japan_3
    ## japan_4
    ## japan_5
    ## japan_11

``` bash
# What if I want the first and the third column?
cut -f1,3 small_text.tsv
```

    ## japan_1  line 1
    ## japan_2  line 2
    ## japan_3  line 3
    ## japan_4  line 4
    ## japan_5  line 5
    ## japan_11 line 11

## `sort` the way you like it

-   In the example above, we extracted the first column of a `tsv` file.
    What if we want to sort that `tsv` file by the 4th column in reverse
    order?

``` bash
# "-k4" tells "sort" we want to sort the tsv file by the 4th column
# "-r" tells "sort" we want the sorting to be in the reverse order
sort -k4 -r small_text.tsv
```

    ## japan_5  iso-seq line 5  5
    ## japan_4  iso-seq line 4  4
    ## japan_3  iso-seq line 3  3
    ## japan_2  iso-seq line 2  2
    ## japan_11 iso-seq line 11 11
    ## japan_1  iso-seq line 1  1

-   Note that `sort` by default sort using all ASCII characters. You
    would have noticed that in the example above, “11” comes before “2”.
    You can tell `sort` to sort numerically by:

``` bash
sort -k4 -r -n small_text.tsv
```

    ## japan_11 iso-seq line 11 11
    ## japan_5  iso-seq line 5  5
    ## japan_4  iso-seq line 4  4
    ## japan_3  iso-seq line 3  3
    ## japan_2  iso-seq line 2  2
    ## japan_1  iso-seq line 1  1

# Some useful command line’s “magic”

## The “wildcard” magic

-   In bash, the character “`*`” is called a wildcard and is often used
    to represent “anything” before or after a string. For example, if I
    want to find any folder that starts with “folder”:

``` bash
# The '-d' parameter specify that we only want to list folders and not files
ls -ld folder*
```

    ## drwxr-xr-x  4 khipinchua  staff  128 Jan 10 18:33 folder
    ## drwxr-xr-x  2 khipinchua  staff   64 Jan 10 21:07 folder_x
    ## drwxr-xr-x  2 khipinchua  staff   64 Jan 10 21:07 folder_y

``` bash
# If you want to remove all folders starting with "folder"
rm -r folder*
```

## The “Tab” magic

-   Try typing `mkd` and press `Tab` on your keyboard.
-   Now try typing `mk` and press `Tab` on your keyboard
-   In general, `Tab` will help you to complete what you are about to
    type if there’s an **unique** command that starts with what you’ve
    typed (e.g. Only `mkdir` starts with `mkd`).
-   When there’s multiple commands that start with what you’ve typed,
    the `bash` shell will provide suggestions on all the possibilities.

## The “pipe” magic

-   We often want to use the output of a command for our next
    command. \* The `pipe` operator is “`|`”s is designed for such
    scenario. E.g. after searching for all the lines that end with 5
    using `grep`, we want to print the final 5 matches with `tail`:

``` bash
# Print the last 5 lines that end with "5" in large_text.txt
grep '5$' large_text.txt | tail -5
```

    ## japan_4955   iso-seq line 4955
    ## japan_4965   iso-seq line 4965
    ## japan_4975   iso-seq line 4975
    ## japan_4985   iso-seq line 4985
    ## japan_4995   iso-seq line 4995

## The “redirect” magic

-   Let’s say after you search for the lines that end with “5”, how do
    you “save” it?
-   In bash, the `redirect` operator is “`>`”:
    -   **Note that `>` will create a new file. If there’s a file with
        the same name of what you want to redirect to, it’ll be
        overwritten!**
    -   **Be very careful not to overwrite any file that you don’t want
        to.**

``` bash
# Look for all the lines that end with 5 and redirect ("save") it in a file
# called "end_with_5.txt"
grep '5$' large_text.txt > end_with_5.txt
ls -lh end_with_5.txt
```

    ## -rw-r--r--  1 khipinchua  staff    14K Jan 18 19:20 end_with_5.txt

``` bash
# Check the first and last 5 lines of the file
head -5 end_with_5.txt
tail -5 end_with_5.txt
```

    ## japan_5  iso-seq line 5
    ## japan_15 iso-seq line 15
    ## japan_25 iso-seq line 25
    ## japan_35 iso-seq line 35
    ## japan_45 iso-seq line 45
    ## japan_4955   iso-seq line 4955
    ## japan_4965   iso-seq line 4965
    ## japan_4975   iso-seq line 4975
    ## japan_4985   iso-seq line 4985
    ## japan_4995   iso-seq line 4995

-   The “`>>`” operator (Two “`>`” joined together) will append (add to
    the end of the file) instead of writing a new file:

``` bash
# Look for first 5 lines that end with 6 and append them to end_with_5.txt
grep -m 5 '6$' large_text.txt >> end_with_5.txt
```

``` bash
# Check the last 10 lines of the file
tail -10 end_with_5.txt
```

    ## japan_4955   iso-seq line 4955
    ## japan_4965   iso-seq line 4965
    ## japan_4975   iso-seq line 4975
    ## japan_4985   iso-seq line 4985
    ## japan_4995   iso-seq line 4995
    ## japan_6  iso-seq line 6
    ## japan_16 iso-seq line 16
    ## japan_26 iso-seq line 26
    ## japan_36 iso-seq line 36
    ## japan_46 iso-seq line 46

## Other useful tools and topics not covered

-   `sed` is a tool that can be used to edit text using `regex`.
-   `awk` is a fantastic tool that can be used to carry out text
    processing.
-   `for` loop and conditional (e.g. `if`) statements.
-   Editing file on command line using text editors such as `vi`, `nano`
    and `emacs`.
    -   Check out `vimtutor` for a quick 30 minutes crash course on
        `vim` (improved version of `vi`)
-   Writing bash scripts (multiple bash commands in a file).
-   Using startup script such as `.bashrc` or `.bash_profile` to setup
    the environment the way you like it.
-   Concept of stdout and stderr.
-   Concept of Unix permission.
-   `scp` and `rsync` to transfer/sync between SSH session and local
    server.
