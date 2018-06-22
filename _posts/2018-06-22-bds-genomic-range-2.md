---

layout: post
title: 'Bioinformatic Data Skills 学习专题(6) Genomic Range之二'
category: reference
tags: [Bioinformatitian]
excerpt_separator: "<!--more-->"

---

# Chapter 9 Working with Range Data (2)


概念回溯
---
 - 为什么要用ranges?
 - 图像画思维：虽然在考虑结构变异的时候我们已经将线性的基因组缩短到了插入、删除、转位和拷贝数目变异等类别，但是更好的方法就是把一维序列当作多位图像来看
 - S4Vectors: S4 implementation of vector-like and list-like objects, http://www.bioconductor.org/packages/release/bioc/html/S4Vectors.html

<!--more-->

## 6. Run Length Encoding and Views
除了可以表示特定位置的核酸序列之外，一个区间也可以有其他含义，每个碱基位置上可以有不同的数值含义，比如：
 - Coverage覆盖度，即碱基上reads的总数
 - Conservation tracks，用于表示某碱基在不同物种间的的进化保守性（可以用phastCons计算）
 - 以人群为单位，计算某特定位置碱基的多样性（哈哈，没错就是SNP）
当然还有很多其他的信息，但是作者在这张主要科普coverage的计算，从序列数据中获得区间信息，以及一个工具view。

### 6.1 Run-length encoding and coverage()
 由于coverage数据通常都比较连续延绵（run），IRanges就用了一个名为`run-length encoding`的算法把值一样的位点重叠起来算作一个'run'（有没有想到一种数据格式？答案见文末，也可以参考我们生信菜鸟团公众号的“数据格式专题”栏目）。这样就可以把分辨率从单碱基缩小到若干碱基乃至一个很大的区域，这就大大节省了存储空间。打个比方就是把一条线很有规律地折叠起来，数值一样的点就算一个run(每个run长度可能不一样，但碱基上的覆盖度必然一样)，像一个个山峰：
![mountain figure]()

#### 6.1.1
首先用`Rle`将包含单碱基覆盖度信息__序列转换为一个run__的信息

```
> x <- as.integer(c(4, 4, 4, 3, 3, 2, 1, 1, 1, 1, 1, 0, 0, 0,
0, 0, 0, 0, 1, 1, 1, 4, 4, 4, 4, 4, 4, 4))
> xrle <- Rle(x)
> xrle
integer-Rle of length 28 with 7 runs
Lengths: 3 2 1 5 7 3 7
Values : 4 3 2 1 0 1 4
```

由于xrle本质上也是IRange的子类，所以同样地，我们也可以对压缩好的数据`xrle`进行IRanges的常规操作：
```
> as.vector(xrle)
[1] 4 4 4 3 3 2 1 1 1 1 1 0 0 0 0 0 0 0 1 1 1 4 4 4 4 4 4 4
> xrle/2  ## 注意runlength不变，值减少为原来的一半
numeric-Rle of length 28 with 7 runs
Lengths: 3 2 1 5 7 3 7
Values : 2 1.5 1 0.5 0 0.5 2
> xrle[xrle > 3]
> runLength(xrle) ## 获得每个run的长度  
[1] 3 2 1 5 7 3 7
> runValue(xrle) ## 获得每个run里碱基的覆盖度
[1] 4 3 2 1 0 1 4
```

#### 6.1.2
`rle`能直接压缩一条序列上的信息，但是如何得到一段序列上若干reads的覆盖度信息呢？

另一种更接地气的问法就是“如何从原始测序数据获得一段特定序列上run-length格式的coverage数据？”。答案如下，关键是`coverage`这个函数。同时IRanges也提供了统计一段区域上run信息的方法，参考第三段代码。

![](/home/whh/pictures/shutter_igv_scntb/bio-0621-012.png)
```
# 1. 造reads，获得含有coverage信息的序列
> set.seed(0)
> rngs <- IRanges(start=sample(seq_len(60), 10), width=7) ## 模拟70条长度为7的序列
> names(rngs)[9] <- "A" # label one range for examples later
> rngs_cov <- coverage(rngs) ## coverage提取reads覆盖度信息
> rngs_cov
integer-Rle of length 63 with 18 runs
Lengths: 11 4 3 3 1 6 4 2 5 2 7 2 3 3 1 3 2 1
Values : 0 1 2 1 2 1 0 1 2 1 0 1 2 3 4 3 2 1
# 这是一个长度为63bp的序列，由18个run组成，参考上图，可以发现有18条横线

# 2. 同样可以取子集
> rngs_cov > 3 # where is coverage greater than 3?
logical-Rle of length 63 with 3 runs
Lengths: 56 1 6
Values : FALSE TRUE FALSE
> rngs_cov[as.vector(rngs_cov) > 3] # extract the depths that are greater than 3
integer-Rle of length 1 with 1 run
Lengths: 1
Values : 4

# 3. 取出某个区域的run信息

> rngs_cov[rngs['A']]  #step1中命名为A一个reads
integer-Rle of length 7 with 2 runs
Lengths: 5 2
Values : 2 1

# 4. 看看这个reads周围的特征，比如丰度之类
> mean(rngs_cov[rngs['A']])  
[1] 1.714286
```
__小结-Coverage节__

很多生物学的问题追根溯源可以归结为遗传和表观遗传问题，但他们都离不开二级结构的特征分析，我们可以问一些问题，比如发生遗传突变的位置是不是少了某些蛋白质的结合？是不是突变的序列导致的？那么这些序列有什么特征呢？GC content/nucleotide diversity等等都是值得追问的；除了探索为止序列（测序结果），我们还可以看实验变量带来的已知序列上的特征改变，比如重复元件、蛋白编码区、低重组区域上的甲基化水平、组蛋白修饰水平或者染色质开放程度的改变。关键还是要探究变量可能带来的影响。
一旦我们把改变量化到区间或者序列上，分辨率降低，计算因此也变得相对高效。_GenomicRanges也提供相类似的功能。_


### 6.2 Going from run-length encoded sequences to ranges with slice()
`Rle`可以把区域信息pileup起来，pileup的数据需要做一定的过滤，比如碱基的coverge至少要为某个值，这就需要用`slice`函数去处理获得的run信息。`slice`很形象，可以想象在峰图上画一条水平线，保留在水平线以上的区域，因此获得的信息多是连续的run。被slice过的rle称为`view`，属于一个序列的一部分，特征是存在一个个峰。


```
> min_cov2 <- slice(rngs_cov, lower=2)
> min_cov2
Views on a 63-length Rle subject
views:
start end width
[1] 16 18 3 [2 2 2]
[2] 22 22 1 [2]
[3] 35 39 5 [2 2 2 2 2]
[4] 51 62 12 [2 2 2 3 3 3 4 3 3 3 2 2]
```

### 6.3 Advanced IRanges: Views
我们使用`slice`获得`rngs_cov`中run的子集或者并集，得到了一个覆盖度大于一定值的`view`。某种意义上，`view`是的`rngs_cov`子类，这个子类继承了父类的方法，但是在`view`中，可用的方法前面都要加一个view，比如`viewMaxs`, `viewMeans`, `viewApply`。

```
> viewMeans(min_cov2) # 统计coverage>2连续区间上覆盖reads的平均值
[1] 2.000000 2.000000 2.000000 2.666667
> viewMaxs(min_cov2) # 统计coverage>2连续区间上覆盖reads数的最大值
[1] 2 2 2 4
> viewApply(min_cov2, median)
[1] 2 2 2 3
```

除了根据覆盖度可以筛选特定区域，我们也可以把一个区域分成若干个__window/bin__，统计每个window信息

```
> length(rngs_cov)
[1] 63
> bwidth <- 5L # 设定bin宽度为5bp
> end <- bwidth * floor(length(rngs_cov) / bwidth) # 获得序列最后一个位置信息
> windows <- IRanges(start=seq(1, end, bwidth), width=bwidth) # 根据start, end和binWidth制定windows
> head(windows)
IRanges of length 6
start end width
[1] 1 5 5
[2] 6 10 5
[3] 11 15 5
[4] 16 20 5
[5] 21 25 5
[6] 26 30 5
> cov_by_wnd <- Views(rngs_cov, windows) # 用Views获得rngs_cov上我们指定的windows上的信息
> head(cov_by_wnd)
Views on a 63-length Rle subject
views:
start end width
[1] 1 5 5 [0 0 0 0 0]
[2] 6 10 5 [0 0 0 0 0]
[3] 11 15 5 [0 1 1 1 1]
[4] 16 20 5 [2 2 2 1 1]
[5] 21 25 5 [1 2 1 1 1]
[6] 26 30 5 [1 1 1 0 0]
> viewMeans(cov_by_wnd) #就可以得到均等区间上的平均信号啦！
[1] 0.0 0.0 0.8 1.6 1.2 0.6 0.8 1.8 0.2 0.4 2.4 3.2

```



### 7. Storing Genomic Ranges with GenomicRanges
`GenomicRanges`在`IRanges`基础上加了一类`GRanges`对象用于储存序列信息，它在`IRanges`基础上加入了其他两类信息用来指明基因组信息：序列名称比如染色体号以及正负链信息。

在指明序列的基础上，我们需要用`metadata columns`指明GRanges上的除了序列信息之外的额外信息，比如gc含量，覆盖度。所有metadata数据都存储在`DataFrame`里，区别于R里的data.frame，支持更多了列类型，比如：
 - run-length encoded vectors
 - identifiers and names: 转录本、snp信息、外显子名称等等
 - annotation data： GC 含量、重复序列信息、保守性打分等等
 - __假如是比对的reads信息，还可以存储Q30和gap数啊，真是妙用!__ ![星星眼]()

"the union of genomic location with any type of data is what makes GRanges so powerful"
 ---


```
> library(GenomicRanges)
> gr <- GRanges(seqname=c("chr1", "chr1", "chr2", "chr3"),
+ ranges=IRanges(start=5:8, width=10),
+ strand=c("+", "-", "-", "+"))
> gr
GRanges object with 4 ranges and 0 metadata columns:
      seqnames    ranges strand
         <Rle> <IRanges>  <Rle>
  [1]     chr1   [5, 14]      +
  [2]     chr1   [6, 15]      -
  [3]     chr2   [7, 16]      -
  [4]     chr3   [8, 17]      +
  -------
  seqinfo: 3 sequences from an unspecified genome; no seqlengths
```


```
#2. 再加点料，比如gc数目, 注意gc是metadata哦，所以有`|`分割
> gr <- GRanges(seqname=c("chr1", "chr1", "chr2", "chr3"),
  ranges=IRanges(start=5:8, width=10),
  strand=c("+", "-", "-", "+"), gc=round(runif(4), 3))
> gr
GRanges object with 4 ranges and 1 metadata column:
      seqnames    ranges strand |        gc
         <Rle> <IRanges>  <Rle> | <numeric>
  [1]     chr1   [5, 14]      + |     0.238
  [2]     chr1   [6, 15]      - |      0.45
  [3]     chr2   [7, 16]      - |     0.236
  [4]     chr3   [8, 17]      + |      0.49
  -------
  seqinfo: 3 sequences from an unspecified genome; no seqlengths

#3. 如何解决“seqinfo: 3 sequences from an unspecified genome; no seqlengths”？

## strategy1
seqlens <- c(chr1=152, chr2=432, chr3=903)
gr <- GRanges(seqname=c("chr1", "chr1", "chr2", "chr3"),
ranges=IRanges(start=5:8, width=10),
strand=c("+", "-", "-", "+"),
gc=round(runif(4), 3),
seqlengths=seqlens)

## strategy2
> seqlengths(gr) <- seqlens # another way to do the same as above

#4.基本操作
start(gr)
end(gr)
width(gr)
seqnames(gr)
strand(gr)
ranges(gr)
length(gr) #4
names(gr) <- letters[1:length(gr)]
start(gr) > 7
table(seqnames(gr)) # 有多少条seq，比如染色体

#5. metadata相关操作,可以计算子集的各种metadata的统计信息
mcols(gr) #metadata的列信息
mcols(gr)$gc
> mcols(gr)$gc #等价
[1] 0.897 0.266 0.372 0.573
> gr$gc
[1] 0.897 0.266 0.372 0.573
> mcols(gr[seqnames(gr) == "chr1"])$gc
[1] 0.897 0.266
> mean(mcols(gr[seqnames(gr) == "chr1"])$gc)
[1] 0.5815
```


### 8. Grouping Data with GRangesList
我们可以使用`GRangesList`或`c()`把`GRanges`对象整合到列表中。`seqnames()`, `start()`, `end()`, `width()`, `ranges()`, `strand()`也都可以用。
```
> gr1 <- GRanges(c("chr1", "chr2"), IRanges(start=c(32, 95), width=c(24, 123)))
> gr2 <- GRanges(c("chr8", "chr2"), IRanges(start=c(27, 12), width=c(42, 34)))
> grl <- GRangesList(gr1, gr2)
> doubled_grl <- c(grl, grl)
> length(doubled_grl)

# 操作grl： 根据染色体信息split grl
> chrs <- c("chr3", "chr1", "chr2", "chr2", "chr3", "chr1")
> gr <- GRanges(chrs, IRanges(sample(1:100, 6, replace=TRUE),
width=sample(3:30, 6, replace=TRUE)))
> head(gr)
GRanges with 6 ranges and 0 metadata columns:
seqnames ranges strand
<Rle> <IRanges> <Rle>
[1] chr3 [90, 93] *
[2] chr1 [27, 34] *
[3] chr2 [38, 44] *
[4] chr2 [58, 79] *
[5] chr3 [91, 103] *
[6] chr1 [21, 44] *
---
seqlengths:
chr3 chr1 chr2
NA NA NA
> gr_split <- split(gr, seqnames(gr))
> gr_split[[1]]
GRanges with 4 ranges and 0 metadata columns:
seqnames ranges strand
<Rle> <IRanges> <Rle>
[1] chr3 [90, 93] *
[2] chr3 [91, 103] *
[3] chr3 [90, 105] *
[4] chr3 [95, 117] *
---
seqlengths:
chr3 chr1 chr2
NA NA NA
> names(gr_split)
[1] "chr3" "chr1" "chr2


# lapply操作 grl
> lapply(gr_split, function(x) order(width(x)))
$chr3
[1] 1 2 3 4
$chr1
[1] 1 2
$chr2
[1] 1 4 2 3
> sapply(gr_split, function(x) min(start(x)))
chr3 chr1 chr2
90 21 38
> sapply(gr_split, length)
chr3 chr1 chr2
4 2 4
> elementLengths(gr_split)
chr3 chr1 chr2
4 2 4
```
### 9. Working with Annotation Data: GenomicFeatures and rtracklayer
这节主要讲了两个包，他们的核心就是GenomicRanges的各种扩展。
 - 一个是`GenomicFeatures`，其工作基础是菜鸟团前期介绍过的一个注释数据库`TranscriptDb`，有各种transcript的信息。
 - 另一个包`rtracklayer`能够导入和导出许多常见的文件类型，BED,WIG,GTF,还能查询和导航UCSC genome Browser rtracklayer包含测序所用BED文件。

*emmm，其实，这两个包我们之前都已经有专门提到过啦，详见前期推送的两个相关专题：
[R专题和Bioconductor注释专题](https://mp.weixin.qq.com/s?__biz=MzUzMTEwODk0Ng==&mid=2247485113&idx=1&sn=828c4938789eb4b22be2ad809c89f44b&chksm=fa46c384cd314a924e118a6d69f319cd4009f5e8b6d22e9eb90d87180a89ea03219a104e75be#rd)，小菜鸟们务必参考一下*

