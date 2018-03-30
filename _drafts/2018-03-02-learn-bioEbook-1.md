---

layout: post
title: 'Learn Bioinformatics Data Skills'
category: codings 
tags: [Reviews, NGS基础, 菜鸟团周推]
excerpt_separator: "<!--more-->"
last_modified_at: 2018-03-02T21:30:31-22:10

---

Start from Part II. Prerequisites: Essential Skills for Getting Started with a Bioinformatics Project

| week | task |
|--------|--------------|
| week1 |  Managing a Bioinformatics Project (2) |

# Setting Up and Managing a Bioinformatics Project
In this chapter, we’ll look at some best practices in organizing your bioinformatics project directories and how to digitally document your work using plain-text Markdown files. We’ll also see why project directory organization isn’t just about being tidy, but is essential to the way by which tasks are automated across large numbers of files (which we routinely do in bioinformatics)

- Project Directories and Directory Structures
- Project Documentation
- Use Directories to Divide Up Your Project into Subprojects
- Organizing Data to Automate File Processing Tasks
- Markdown for Project Notebooks
- Markdown Formatting Basics
- Using Pandoc to Render Markdown to HTML

## 1. Project Directories and Directory Structures
Regardless of the organization scheme, a good bioinformatician will always document everything extensively and use clear filenames that can be parsed by a computer, two points we’ll come to in a bit.

When in doubt, explicit is always better than implicit when it comes to filenames, documentation, and writing code.

__always use relative paths__ 
In contrast, absolute paths rely on your particular user account and directory structures details above the project directory level (not good). Using absolute paths leaves your work less portable
between collaborators and decreases reproducibility.

mkdir data
mkdir data/seqs scripts analysis

This is a sensible project layout scheme. Here, data/ contains all raw and intermediate
data. As we’ll see, data-processing steps are treated as separate subprojects in this
data/ directory. I keep general project-wide scripts in a scripts/ directory. If scripts
contain many files (e.g., multiple Python modules), they should reside in their own
subdirectory. Isolating scripts in their own subdirectory also keeps project directories
tidy while developing these scripts (when they produce test output files).
Bioinformatics projects contain many smaller analyses—for example, analyzing the
quality of your raw sequences, the aligner output, and the final data that will produce
figures and tables for a paper. I prefer keeping these in a separate analysis/ directory,
as it allows collaborators to see these high-level analyses without having to dig deeper
into subproject directories.


## 2. Project Documentation
There’s a vast amount of lurking complexity in bioinformatics work: complex workflows, multiple files, countless program parameters, anddifferent software versions. The best way to prevent this complexity from causingproblems is to document everything extensively. 

 - Document your methods and workflows
This should include full command lines (copied and pasted) that are run through the shell that generate data or intermediate results. Even if you use the default values in software, be sure to write these values down; later versions of the program may use different default values.  

__In general, any command that produces results used in your work needs to be documented somewhere.__
### Track: Where, When, and How
 - Document the origin of all data in your project directory
“Data” doesn’t just refer to your project’s experimental data—it’s any data that programs use to create output. This includes files your collaborators send you from their separate analyses, gene annotation tracks, reference genomes, and so on. 
Need to keep track of where data was downloaded from, who gave it to you,
and any other relevant information.

 - Document when you downloaded data
It’s important to include when the data was downloaded, as the external data
source (such as a website or server) might change in the future. 

 - Record data version information
Many databases have explicit release numbers, version numbers, or names (e.g.,
TAIR10 version of genome annotation for Arabidopsis thaliana, or Wormbase
release WS231 for Caenorhabditis elegans). It’s important to record all version
information in your documentation, including minor version numbers.

 - Describe how you downloaded the data
For example, did you use MySQL to download a set of genes? Or the UCSC
Genome Browser? These details can be useful in tracking down issues like when
data is different between collaborators.

 - Document the versions of the software that you ran
This may seem unimportant, but remember the example from “Reproducible
Research” on page 6 where my colleagues and I traced disagreeing results down
to a single piece of software being updated. These details matter. Good bioinfor‐
matics software usually has a command-line option to return the current version.
Software managed with a version control system such as Git has explicit identifi‐
ers to every version, which can be used to document the precise version you ran
(we’ll learn more about this in Chapter 5). If no version information is available,
a release date, link to the software, and download date will suffice


All of this information is best stored in plain-text README files. Plain text can easily
be read, searched, and edited directly from the command line, making it the perfect
choice for portable and accessible README files. 
Where should you keep your README files? A good approach is to keep README
files in each of your project’s main directories. These README files don’t necessarily
need to be lengthy, but they should at the very least explain what’s in this directory
and how it got there. 
For example, a data/README file would contain metadata about your data files in
the data/ directory. Even if you think you could remember all relevant information
about your data, it’s much easier just to throw it in a README file (and collaborators
won’t have to email you to ask what files are or where they are). 

`touch README data/README`


## 3. Use Directories to Divide Up Your Project into Subprojects
Take-home point is: leverage directories to help stay organized
simplify complex projects

## 4. Organizing Data to Automate File Processing Tasks
consistent file naming schemes
tips:
    - brace expansion "{}": `mkdir -p zmays-snps/{data/seqs,scripts,analysis}`
    - 

## 5. Markdown for Project Notebooks
__plain-text project notebooks__
Keep a project notebook containing detailed information about:
 - the chronology of your computational work
 -  steps you’ve taken, 
 -  information about why you’ve made decisions, and 
 -  of course all pertinent information to reproduce your work. 

## 6. Markdown Formatting Basics
## 7. Using Pandoc to Render Markdown to HTML
Pandoc can convert between a variety of different markup and output formats. Using
Pandoc is very simple—to convert from Markdown to HTML, use the --from mark
down and --to html options and supply your input file as the last argument:
`$ pandoc --from markdown --to html notebook.md > output.html`


# Remember in heart
__Part 1, Ideology: Data Skills for Robust and Reproducible Bioinformatics__
 - Why Bioinformatics? Biology’s Growing Data
 - Learning Data Skills to Learn Bioinformatics
 - New Challenges for Reproducible and Robust Research
 - Reproducible Research
 - Robust Research and the Golden Rule of Bioinformatics
 - Adopting Robust and Reproducible Practices Will Make Your Life Easier, Too
 - Recommendations for Robust Research
 - Pay Attention to Experimental Design
 - Write Code for Humans, Write Data for Computers
 - Let Your Computer Do the Work For You
 - Make Assertions and Be Loud, in Code and in Your Methods
 - Test Code, or Better Yet, Let Code Test Code
 - Use Existing Libraries Whenever Possible
 - Treat Data as Read-Only
 - Spend Time Developing Frequently Used Scripts into Tools
 - Let Data Prove That It’s High Quality
 - Recommendations for Reproducible Research
 - Release Your Code and Data
 - Document Everything
 - Make Figures and Statistics the Results of Scripts
 - Use Code as Documentation
 - Continually Improving Your Bioinformatics Data Skills
