+++
title = "Bash Day 1 mid-day session"
slug = "bash2"
+++

1. Are there any questions on the materials you just watched?
1. Let's go together through the challenges, do some exercises, and debug problems.

Click on a triangle to expand a question:

{{< question num=1 >}}
Relative vs. absolute paths. Using `~` as part of a longer path.
{{< /question >}}

{{< question num=2 >}}
If `pwd` displays `/users/thing`, what will `ls ../backup` display?
{{< figure src="/img/quizDirs.png" >}}
{{< /question >}}

{{< question num=3 >}}
Given the same directory structure, if `pwd` displays `/users/backup`, and `-r` tells `ls` to display things in reverse
order, what command will display:
```
pnas-sub/  pnas-final/  original/
```
{{< /question >}}

{{< question num=4 >}}
What does the command `cd` do if you do not pass it a directory name?
{{< /question >}}

{{< question num=5 >}}
Starting from `/Users/amanda/data/`, which of the following commands could Amanda use to navigate to her home directory,
which is `/Users/amanda`? Mark all correct answers.
{{< /question >}}

{{< question num=6 >}}
Suppose that you created a .txt file in your current directory to contain a list of the statistical tests you will need
to do to analyze your data, and named it `statstics.txt`. After creating and saving this file, you realize you
misspelled the filename! You want to correct the mistake, which of the following commands could you use to do so?
{{< /question >}}

{{< question num=7 >}}
Write simple aliases for safer `mv`, `cp` so that these do not automatically overwrite the target. Hint: use their
manual pages. Where would you store these aliases? (no need to type any answer)
{{< /question >}}

{{< question num=8 >}}
Write simple alias for safer `rm`. (no need to type any answer)
{{< /question >}}

{{< question num=9 >}}
What is the output of the last `ls` command in the sequence shown below?
```sh
$ pwd
/home/jamie/data
$ ls
proteins.dat
$ mkdir recombine
$ mv proteins.dat recombine
$ cp recombine/proteins.dat ../proteins-saved.dat
$ ls
```
{{< /question >}}

{{< question num=10 >}}
Running `ls -F` in `~/Desktop/Shell/Users/nelle/sugars` results in:
```sh
analyzed/  glucose.dat  mannose.dat  sucrose.dat  fructose.dat  maltose.dat  raw/
```
What code would you use to move all the .dat files into the analyzed sub-directory?
{{< /question >}}

{{< question num=11 >}}
In a directory we want to find the 3 files that have the least number of lines. Which command would work for this?
{{< /question >}}

{{< question num=12 >}}
In a directory the command `ls` returns:
```sh
fructose.dat  glucose.dat  sucrose.dat  maltose.txt
```
What would be the output of the following loop?
```sh
for datafile in *.dat
do
  cat $datafile >> sugar.dat
done
```
{{< /question >}}

{{< question num=13 >}}
Using `diff` to compare files and directories. (no need to type any answer)
{{< /question >}}

{{< question num=14 >}}
Discuss brace expansion. Try nested braces. Paste an example that works.

What will this command do:
```sh
touch 2022-May-{0{1..9},{10..30}}.md
```
{{< /question >}}

{{< question num=15 >}}
Use `ps` command to see how many processes you are running on the training cluster. Explore its flags. (no need to type
any answer)
{{< /question >}}

{{< question num=16 >}}
Copy a file to/from the training cluster using either `scp` or `sftp`. (no need to type any answer)
{{< /question >}}

{{< question num=17 >}}
Bring up the manual page on `rsync`, then use it to synchronize a directory from the training cluster. (no need to type
any answer)
{{< /question >}}

{{< question num=18 >}}
Using Unix pipes, write a one-line command to show the name of the longest .pdb file (by the number of lines). Paste
your answer into the chat.
{{< /question >}}

{{< question num=19 >}}
Combine `ls` and `head` and/or `tail` into a one-line command to show three largest files (by the number of bytes) in a
given directory. Paste your answer into the chat.
{{< /question >}}

{{< question num=20 >}}
Write a loop that concatenates all .pdb files in `data-shell/molecules` subdirectory into one file called
`allmolecules.txt`, prepending each fragment with the name of the corresponding .pdb file, and separating different
files with an empty line. Run the loop, make sure it works, bring it up with the "*up arrow*" key and paste into the
chat.
{{< /question >}}

{{< question num=21 >}}
Use Ctrl-C to kill an infinite (or very long) loop. (no need to type any answer)
{{< /question >}}

{{< question num=22 >}}
Play with variables and their values. Change the prompt, e.g. `PS1="\u@\h \w> "`. (no need to type any answer)
{{< /question >}}

{{< question num=23 >}}
What will the command `echo directoryName/*` do? Try answering without running it. How is this output different from `ls
directoryName` and `ls directoryName/*`?
{{< /question >}}

{{< question num=24 >}}
What will the loop `for i in hello 1 2 * bye; do echo $i; done` print? Try answering without running the loop.
{{< /question >}}

{{< question num=25 >}}
Pasting variables into strings:
```sh
var="sun"
echo $varshine
echo ${var}shine
echo "$var"shine
```
(no need to type any answer)
{{< /question >}}

{{< question num=26 >}}
Create a loop that writes into 10 files `chapter01.md`, `chapter02.md`, ..., `chapter10.md`. Each file should contain
chapter-specific lines, e.g. `chapter05.md` will contain exactly these lines:
```sh
## Chapter 05
This is the beginning of Chapter 05.
Content will go here.
This is the end of Chapter 05.
```
(no need to type any answer)
{{< /question >}}

{{< question num=27 >}}
Redirection `1>` and `2>` and `/dev/null` (no need to type any answer)
{{< /question >}}

{{< question num=28 >}}
`;` vs. `&&` separators, e.g. `mkdirr tmp; cd tmp` (no need to type any answer)
{{< /question >}}

{{< question num=29 >}}
Variable manipulation:
```sh
myvar="hello"
echo $myvar
echo ${myvar:offset}
echo ${myvar:offset:length}
echo ${myvar:2:3}    # 3 characters starting from character 2
echo ${myvar/l/L}    # replace the first match of a pattern
echo ${myvar//l/L}   # replace all matches of a pattern
```
(no need to type any answer)
{{< /question >}}

{{< question num=30 >}}
Using knowledge from the previous question, write a loop to replace spaces to underscores in all file names in the
current directory.
```sh
touch hello "first phrase" "second phrase" "good morning, everyone"
ls -l
ls *\*
```
{{< /question >}}

{{< question num=31 >}}
Why `mv *.txt *.bak` does not work? Write a loop to rename all .txt files to .bak files. There are several solutions for
changing a file extension inside a loop you know by now.
{{< /question >}}

{{< question num=32 >}}
Are there questions on any of the topics that we covered today? You can type your question into the chat, ask via audio
(unmute), or raise your hand in Zoom.
{{< /question >}}



<!-- {{< solution >}} -->
<!-- ```sh -->
<!-- function countfiles() { -->
<!--     if [ $# -eq 0 ]; then -->
<!--         echo "No arguments given. Usage: countfiles dir1 dir2 ..." -->
<!--         return 1 -->
<!--     fi -->
<!--     for dir in $@; do -->
<!--         echo in $dir we found $(find $dir -type f | wc -l) files -->
<!--     done -->
<!-- } -->
<!-- ``` -->
<!-- {{< /solution >}} -->
