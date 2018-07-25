---

layout: post
title: '学bds之Chapter 10 Working with Sequence Data'
category: reference
tags: [Bioinformatitian]
excerpt_separator: "<!--more-->"

---



## 前言
“生物信息数据处理中，处理各种数据格式属于关键技能之一，一些一开始人为设定的文件格式会随着时间成为标准的数据格式。” ——Peter Cock
这章讲了一些核酸序列的数据格式的优缺点，以及处理这些数据的工具。这章比较短，但有非常重要的一点需要说明：要非常小心人为设定的数据格式所可能出现的常见陷阱。一些细节上的差错（比如文件类型）可能会让我们花大量的时间和精力去查错，所以早期对细节的关注非常重要。

<!--more-->
# CHAPTER 10: Working with Sequence Data
## 前言
“生物信息数据处理中，处理各种数据格式属于关键技能之一，一些一开始人为设定的文件格式会随着时间成为标准的数据格式。” ——Peter Cock
这章讲了一些核酸序列的数据格式的优缺点，以及处理这些数据的工具。这章比较短，但有非常重要的一点需要说明：要非常小心人为设定的数据格式所可能出现的常见陷阱。一些细节上的差错（比如文件类型）可能会让我们花大量的时间和精力去查错，所以早期对细节的关注非常重要。

## FASTA 格式
fasta格式一般在参考序列里出现，不要求出现每个碱基的质量分数（因为本身就是参考），比如CDS，转录本序列等等。数据格式由两部分组成：
 - 描述的信息文件，以`>`开头，后跟序列名称
 - 序列数据，A/T/G/C

因为只要求两类结构，所以其格式相当多变，通常情况下我们都需要从已有的数据库中获得fasta文件以保持数据的整齐性。
```sh
>ENSMUSG00000020122|ENSMUST00000138518
CCCTCCTATCATGCTGTCAGTGTATCTCTAAATAGCACTCTCAACCCCCGTGAACTTGGT
TATTAAAAACATGCCCAAAGTCTGGGAGCCAGGGCTGCAGGGAAATACCACAGCCTCAGT
TCATCAAAACAGTTCATTGCCCAAAATGTTCTCAGCTGCAGCTTTCATGAGGTAACTCCA
GGGCCCACCTGTTCTCTGGT
```

获得相应数据后我们还得清洗一下，否则很容易出现下面这种情况：
```sh
>ENSMUSG00000020122|ENSMUST00000138518
> ENSMUSG00000020122|ENSMUST00000125984
>ENSMUSG00000020122|ENSMUST00000125984|epidermal growth factor receptor
>ENSMUSG00000020122|ENSMUST00000125984|Egfr
>ENSMUSG00000020122|ENSMUST00000125984|11|ENSFM00410000138465
```

一般而言，我们都需要自己写一个特定脚本转换某格式到所需的固定格式，作者建议序列名称的结构通常可以是`identifier + comment`的格式，比如`>gene_00284728 length=231;type=dna`，一目了然。

## FASTQ格式
FASTQ在FASTA的基础上加了每个碱基的质量分数:
![](http://owxb9z5ea.bkt.clouddn.com/18-7-20/61053266.jpg)
 - 1. description line, @ 开头
 - 2. 序列数据
 - 3. +开头的行代表着序列结束
 - 4. 序列每个碱基质量分数，和2一一对应

## 核苷酸编码标准：IUPAC
 在知道这些数据格式之后，我们还需要看看数据的核乾酸编码和碱基质量读数的标准是什么。大写的ATCG代表各碱基，小写则用来表示repeats或low complexity sequence（可以用repeatMasker和Tandem Repeats Finder找）。根据IUPAC标准，歧义核酸序列可以用多个N来表示。

## 碱基质量 (Base Quality)
一个碱基对应的质量分数用单个ASCII字符编码，它们代表0～127中的整数。但实际上，由于其中编码的某些字符不可见，所以最后33～126才被用来表示质量。我们可以直接用编程语言的内置函数把ASCII字符转转换成10进制的数，比如python用`ord()`和`chr()`。但是由于不同测序平台用不同的字符表示质量，我们不能直接用10进制的数解读质量，还需要平台信息，不同平台由不同的质量分数范围。常见的有Sange平台的PHRED33算法范围, Solexa平台的算法, Illumina平台的PHRED64（不知该庆幸还是悲哀，目前Illumina已呈现出一家独大的趋势，基本上我们就可以用它的字符来解码了）。PHRED来源是Phil Green，`P = 10-Q/10` <-> `Q = -10 log10P`。

## 实例

### 1. 检测并去除低质量读数的碱基
通常在序列前部质量都比较好，随着荧光强度的增强，噪音越来越大，因此碱基的准确性越来越低。所以，为了获得高质量的reads通常哦我们可以取出末尾的碱基，但是当深度不够的时候，就需要挑出个别质量差的了。文中给出`qrqc`这个R包作为可视化的示例，`sickle` 和`seqtk`被用来处理序列。

```sh
$ sickle se -f untreated1_chr4.fq -t sanger -o untreated1_chr4_sickle.fq
FastQ records kept: 202866
FastQ records discarded: 1489

$ seqtk trimfq untreated1_chr4.fq > untreated1_chr4_trimfq.fq
```

```R
# trim_qual.R -- explore base qualities before and after trimming
library(qrqc)
# FASTQ files
fqfiles <- c(none="untreated1_chr4.fq",
sickle="untreated1_chr4_sickle.fq",
trimfq="untreated1_chr4_trimfq.fq")
# Load each file in, using qrqc's readSeqFile
# We only need qualities, so we turn off some of
# readSeqFile's other features.
seq_info <- lapply(fqfiles, function(file) {
readSeqFile(file, hash=FALSE, kmer=FALSE)
})
# Extract the qualities as dataframe, and append
# a column of which trimmer (or none) was used. This
# is used in later plots.
quals <- mapply(function(sfq, name) {
qs <- getQual(sfq)
qs$trimmer <- name
qs
}, seq_info, names(fqfiles), SIMPLIFY=FALSE)

# Combine separate dataframes in a list into single dataframe
d <- do.call(rbind, quals)
# Visualize qualities
p1 <- ggplot(d) + geom_line(aes(x=position, y=mean, linetype=trimmer))
p1 <- p1 + ylab("mean quality (sanger)") + theme_bw()
print(p1)
```
![](http://owxb9z5ea.bkt.clouddn.com/18-7-25/29051935.jpg)

```R
# Use qrqc's qualPlot with list produces panel plots
# Only shows 10% to 90% quantiles and lowess curve
p2 <- qualPlot(seq_info, quartile.color=NULL, mean.color=NULL) + theme_bw()
p2 <- p2 + scale_y_continuous("quality (sanger)")
print(p2)
```
![](http://owxb9z5ea.bkt.clouddn.com/18-7-25/4315858.jpg)
### 2. 统计碱基数目
这里作者说道我们虽然可以自己写代码统计，但是可移植性不强，所以还是用那些成熟的软件去做，比如[Heng Li写的软件包系列](https://github.com/lh3/readfq)。
他也自己写了个脚本作为例子：
```py
#!/usr/bin/env python
# nuccount.py -- tally nucleotides in a file
import sys
from collections import Counter
from readfq import readfq
IUPAC_BASES = "ACGTRYSWKMBDHVN-."
# intialize counter
counts = Counter()
for name, seq, qual in readfq(sys.stdin):
# for each sequence entry, add all its bases to the counter
counts.update(seq.upper())
# print the results
for base in IUPAC_BASES:
print base + "\t" + str(counts[base])
```

### 3. Indexed files are ubiquitous in bioinformatics
很多时候我们需要fasta文件大量的随机区域，但是假如按照顺序把所有染色体都读入内存肯定不现实，所以index的存在就很有必要。用samtools就能实现功能：
`samtools faidx Mus_musculus.GRCm38.75.dna.chromosome.8.fa`，会创建一个fai结尾的经过index的文件，其实和chrom.sizes很像，就是所有染色体号，长度的bed文件。
为什么index会使查询变快呢？就是坐标和坐标的对应关系，fai文件记录了排好序的染色体位置信息，同时也有对应二进制文件的具体序列信息，所以通过fai查阅就能获得对应的详细的信息了——用一个土气的形容，应该叫做“区块化”吧，哈哈。


## 下节
虽然以前的专题有集中介绍，这节从单条数据的格式和简单处理，让读者从练习中更了解格式。因此下期主要要讲的就是原始数据比对好的结果：SAM/BAM 文件：
 - 格式：header/section/flags/CIGAR/Mapping quality
 - 相关的处理工具: samtools
 - 用IGV可视化
 - 用Pysam自己做SAM/BAM处理工具
