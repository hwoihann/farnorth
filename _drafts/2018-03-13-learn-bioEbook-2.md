---

layout: post
title: "week#2  Basic shell and remote machines (3&7, 4)"
category: codings 
tags: [Reviews, NGS基础, 菜鸟团周推]
excerpt_separator: "<!--more-->"
last_modified_at: 2018-03-13T21:20:31-22:20

---
# Chapter3
1. Remedial concepts: treams, redirection, pipes, working with running programs, and command substitution.
2. Why the Unix shell has such a prominent role in how we do modern bioinformatics.
3. Modularity, interfacing and text streams
    - easier to spot errors
    - Alternative methods and approaches
    - Combination of cools
    - Reusable!
<!--more-->

Bourne-again shell, or bash. Bash is widely available and the default shell on operating systems like Apple’s OS X and Ubuntu Linux. 
`echo $0` / `echo $SHELL`
Z shell has more advanced features (e.g., better autocomplete) that come in handy in bioinformatics.
> resources about Z shell in this chapter’s README file on GitHub.

wildcards n. 通配符
primitives n. [计] 基元（primitive的复数）；原始事物；基本体 
Almighty adj. 全能的；有无限权力的
caret n. 脱字符号；插入符号
on the fly na. 〈非正式〉匆匆忙忙地；在空中；飞行中；

Unix shell primitives that allow us to build complex programs from simple parts: streams, redirection, pipes, working with processes, and command substitution. 


# 1. Working with Streams and Redirection
Text streams allow us to do processing on a stream of data rather than holding it all in memory


## 1.1. Redirecting Standard Out to a File

## 1.2. Redirecting Standard Error
In practice, we often want to redirect the standard error stream to a file so messages, errors, and warnings are logged to a file we can check later
` 2> listing.stderr`

```
$ ls -l tb1.fasta leafy1.fasta > listing.txt 2> listing.stderr
$ cat listing.txt
-rw-r--r-- 1 vinceb staff 152 Jan 20 21:24 tb1.fasta
$ cat listing.stderr
ls: leafy1.fasta: No such file or directory
```

## 1.3. Using Standard Input Redirection
Though standard input redirection is less common than >, >>, and 2>, it is still occasionally useful:
`$ program < inputfile > outputfile`  =  `cat inputfile | program > outputfile`

Modern disks are orders of magnitude slower than memory!

# 2. The Almighty Unix Pipe: Speed and Beauty in One
## 2.1. Pipes in Action: Creating Simple Programs with Grep and Pipes

```
grep -v "^>" tb1.fasta | \
grep --color -i "[^ATCG]"
CCCCAAAGACGGACCAATCCAGCAGCTTCTACTGCTAYCCATGCTCCCCTCCCTTCGCCGCCGCCGACGC
```
invert the matching lines with the grep option -v
ignore case with -i
add grep’s --color option to color the matching nonnucleotide characters.
The backslash (\) character simply means that we’re continuing the command on the next line, and is used above to improve readability


## 2.2. Combining Pipes and Redirection
`$ program1 2>&1 | grep "error"`
The 2>&1 operator is what redirects standard error to the standard output stream.

## 2.3. Even More Redirection: A tee in Your Pipe
However, we do occasionally need to write intermediate files to disk in Unix pipelines. These intermediate files can be useful when debugging a pipeline or when you wish to store intermediate files for steps that take a long time to complete. 

Like a plumber’s T joint, the Unix program tee diverts a copy of your pipeline’s standard output stream to an intermediate file while still passing it through its standard output: 
`$ program1 input.txt | tee intermediate-file.txt | program2 > results.txt`

# 3. Managing and Interacting with Processes
how to work with and manage processes from the Unix shell. 
Basics of manipulating processes: 
 - running and managing processes in the background,
 - killing errant processes, and 
 - checking process exit status

## 3.1. Background Processes
We can tell the Unix shell to run a program in the background by appending an
ampersand (&) to the end of our command. For example:
```
$ program1 input.txt > results.txt &
[1] 26577
```
The number returned by the shell is the process ID or PID of program1. This is a
unique ID that allows you to identify and check the status of program1 later on. We
can check what processes we have running in the background with jobs:
```
$ jobs
[1]+ Running program1 input.txt > results.txt
```

To bring a background process into the foreground again, we can use `fg` (for fore‐
ground). 
To return a specific background job to the foreground, use fg %<num> where <num> is its number in the job list. If we wanted to return program1 to the foreground, both fg and fg %1 would do the same thing, as there’s only one background process:


## 3.2. Killing Processes

## 3.3. Exit Status: How to Programmatically Tell Whether Your Command Worked
$ program1 input.txt > results.txt
$ echo $?
0
Exit statuses are incredibly useful because they allow us to programmatically chain
commands together in the shell.


Using the || operator, we can have the shell execute a command only if the previous
command has failed (exited with a nonzero status). This is useful for warning mes‐
sages:
```
$ program1 input.txt > intermediate-results.txt || \
echo "warning: an error occurred
```


## 3.4. Command Substitution

```
$ grep -c '^>' input.fasta
416
$ echo "There are $(grep -c '^>' input.fasta) entries in my FASTA file."
There are 416 entries in my FASTA file.
```

`$ mkdir results-$(date +%F)`
```
alias mkpr="mkdir -p {data/seqs,scripts,analysis}"
alias gitday="date +%F"
alias today="date +%Y%m%d"
alias wkday='echo "week"$(date +%U)_day`expr $(date +%w) + 1`'

```



Tips:
 - File descriptor
    All open files (including streams) on Unix systems are assigned a unique integer known as a file descriptor. 
>Unix’s three standard streams—standard input (which we’ll see in a bit), stan‐
dard output, and standard error—are given the file descriptors 0, 1, and 2, respectively. 

 -    “fake” disk
    (known as a pseudodevice) to redirect unwanted output to: `/dev/null`. Output written to /dev/null disappears, which is why it’s sometimes jokingly referred to as a “blackhole” by nerds.

 - The Golden Rule of Bioinformatics is to not trust your tools or data. This skepticism requires constant sanity checking of intermediate results, which ensures your methods aren’t biasing your data, or problems in your data aren’t being exacerbated by your methods.

 - Lowercase characters are often used to indicate masked repeat or low-complexity sequences
 
 - Alias today="date +%F"
 -  If you’re running a clever one-liner over and over again, use add alias to add it to your ~/.bashrc (or ~/.profile if on OS X). alias simply aliases your command to a shorter name alias. For example, if you always create project directories with the same directory structure, add a line like the following: 
 -  alias mkpr="mkdir -p {data/seqs,scripts,analysis}" For small stuff like this, there’s no point writing more complex scripts; adopt the Unix way and keep it simple. Another example is that we could alias our `date +%F` command to today: 
        `alias today="date +%F` 
        Now, entering mkdir results-$(today) will create a dated results directory 

alias today="date +%Y%m%d"
alias week="date +%U"
alias wkday='echo "week"$(date +%U)_day`expr $(date +%w) + 1`'

 expr `date +%w` + 1



 # Chapter 4: Working with Remote Machines

 ## Tips
 Storing Your Frequent SSH Hosts
 Bioinformaticians are constantly having to SSH to servers, and typ‐
 ing out IP addresses or long domain names can become quite tedi‐
 ous. It’s also burdensome to remember and type out additional
 details like the remote server’s port or your remote username. The
 developers behind SSH created a clever alternative: the SSH config
 file. SSH config files store details about hosts you frequently con‐
 nect to. This file is easy to create, and hosts stored in this file work
 not only with ssh, but also with two programs we’ll learn about in
 Chapter 6: scp and rsync.
 
 To create a file, just create and edit the file at ~/.ssh/config. Each
 entry takes the following form:
 ```
 Host bio_serv
 HostName 192.168.237.42
 User cdarwin
 Port 50453
 ```
 You won’t need to specify Port and User unless these differ from the remote host’s defaults. With this file saved, you can SSH into  192.168.236.42 using the alias `ssh bio_serv` rather than typing out `ssh -p 50453 cdarwin@192.169.237.42`

 ## Contents

 ### 4.1. Connecting to Remote Machines with SSH

 ### 4.2 Quick Authentication with SSH Keys
 ### 4.2 Maintaining Long-Running Jobs with nohup and tmux



 # Chapter 7: Unix Data Tools

 ## head and tail
 We can also use tail to remove the header of a file. Normally the -n argument speci‐
fies how many of the last lines of a file to include, but if -n is given a number x pre‐
ceded with a + sign (e.g., +x), tail will start from the xth line. So to chop off a header,
we start from the second line with -n +2. Here, we’ll use the command seq to gener‐
ate a file of 3 numbers, and chop of the first line:
```
$ seq 3 > nums.txt
$ cat nums.txt
1 2 3 $
tail -n +2 nums.txt
2 3
```

(head -n 2; tail -n 2) < Mus_musculus.GRCm38.75_chr1.bed
This is a useful trick, but it’s a bit long to type. To keep it handy, we can create a short‐
cut in your shell configuration file, which is either ~/.bashrc or ~/.profile






