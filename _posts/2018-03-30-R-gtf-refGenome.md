
R专题：使用R读取，处理GTF格式的文件.md

## 前言：小白快问快答
__Q1：GTF是个啥？__

见下表和以前的推文：
[NGS数据格式之gff/gtf](https://mp.weixin.qq.com/s?__biz=MzUzMTEwODk0Ng==&mid=2247484320&idx=1&sn=f7c9e7ee096f636449aa5137af7b830d&chksm=fa46c69dcd314f8bef00af84e87769768eb629b370d5af626c9e1ea74a96d1d4210006b9d5ba&mpshare=1&scene=1&srcid=0330YCwz36p9lpcBNZehFzTq&pass_ticket=CKS4DzU20h7YLgPJXblylkaCCQHn%2BV6Pmwb9awL4JCeD2UuddEEkOfaa%2FBUesum2#rd)

![](http://owxb9z5ea.bkt.clouddn.com/18-3-30/46681628.jpg)



__Q2： 为什么要在R里面倒腾GTF？__
为了生信小白对此类数据格式的快速入门+R的学习！


## 学习资料准备
#### 1. __R包__： refGenome
本文基本出自该包官方文档说明：
https://cran.r-project.org/web/packages/refGenome/vignettes/refGenome.pdf

![](http://owxb9z5ea.bkt.clouddn.com/18-3-30/98353593.jpg)


#### 2. 一个gtf文件

```
wget -c ftp://ftp.ensembl.org/pub/release-91/gtf/mus_musculus/Mus_musculus.GRCm38.91.gtf.gz
gunzipMus_musculus.GRCm38.91.gtf.gz
 cat /nethome/Shared/annotation/genomes/GRCm38/Mus_musculus.GRCm38.91.gtf | grep 'gene_name "Xkr4";' 
 cat /nethome/Shared/annotation/genomes/GRCm38/Mus_musculus.GRCm38.91.gtf | grep -v "^#" | wc -l
1768660
1 ensembl_havana CDS 3216025 3216968 . - 2 gene_id "ENSMUSG00000051951"; gene_version "5"; transcript_id "ENSMUST00000070533"; transcript_version "4"; exon_number "3"; gene_name "Xkr4"; gene_source "ensembl_havana"; gene_biotype "protein_coding"; havana_gene "OTTMUSG00000026353"; havana_gene_version "2"; transcript_name "Xkr4-201"; transcript_source "ensembl_havana"; transcript_biotype "protein_coding"; tag "CCDS"; ccds_id "CCDS14803"; havana_transcript "OTTMUST00000065166"; havana_transcript_version "1"; protein_id "ENSMUSP00000070648"; protein_version "4"; tag "basic"; transcript_support_level "1";
```

还是再简单的了解一下它的内容（比较简单，不翻译了:) ）：

1. seqname    The name of the sequence; must be a chromosome or scaffold.
2. source    The program that generated this feature.这个gtf文件中gene的source主要来自于：
    havana
    insdc
    mirbase
    ensembl
    ensembl_havana

3. feature    The name of this type of feature, e.g. "CDS", "start_codon", "stop_codon", and "exon"
4. start    The starting position of the feature in the sequence; the first base is numbered 1.
5. end    The ending position of the feature (inclusive).
6. score    A score between 0 and 1000.
7. strand    Valid entries include "+", "-", or ".".
8. frame    If the feature is a coding exon, frame should be a number between 0-2 that represents the reading frame of the first base. If the feature is not a coding exon, the value should be ".".

然后挑一个顺眼的基因看看它的feature和frame，果然和描述的一致，继续往下看。
```
awk 'BEGIN{FS="\t|;"} ($0  ~/(gene_name "Xkr4")/ ) {print $3,$8}' GRCm38/Mus_musculus.GRCm38.91.gtf
    gene .
    transcript .
    exon .
    exon .
    transcript .
    exon .
    exon .
    transcript .
    exon .
    CDS 0
    start_codon 0
    exon .
    CDS 1
    exon .
    CDS 2
    stop_codon 0
    five_prime_utr .
    three_prime_utr .
```

\\9. group    All lines with the same group are linked together into a single item.

    gene_id "ENSMUSG00000051951"
    gene_id "ENSMUSG00000051951"
    gene_id "ENSMUSG00000051951"
    gene_id "ENSMUSG00000051951"
    gene_id "ENSMUSG00000051951"
    gene_id "ENSMUSG00000051951"
    gene_id "ENSMUSG00000051951"
    gene_id "ENSMUSG00000051951"
    gene_id "ENSMUSG00000051951"
    gene_id "ENSMUSG00000051951"
    gene_id "ENSMUSG00000051951"
    gene_id "ENSMUSG00000051951"
    gene_id "ENSMUSG00000051951"
    gene_id "ENSMUSG00000051951"
    gene_id "ENSMUSG00000051951"
    gene_id "ENSMUSG00000051951"
    gene_id "ENSMUSG00000051951"
    gene_id "ENSMUSG00000051951"

这个single item即指基因id了，一共有53000多个ensmusg。


#### 3. _ 其他的(可跳过，非重点)_ 
 - S4 class：各种annotation相关R包的基础，有兴趣可参看http://www.bioconductor.org/help/course-materials/2010/AdvancedR/S4InBioconductor.pdf
 - 正则表达式 

## 正文：refGenome使用简介

这个包主要适用于gtf文件（比如Ensembl和UCSC数据）的读取和操作，基于S4 class，可以分成_refGenome_，_refExons_ 和 _refJunctions_三个子类。每一个类别都对应着Ensemble或UCSC的定义类型（顾名思义，genome、exon……）。本文主要讲讲相对简单的genome和exon对象的相关操作。

### 1. 安装
安装，读取gtf，可以看到loading的信息。

```
install.packages("refGenome") 
library(refGenome) 

# create ensemblGenome object for storing Ensembl genomic annotation data
ens <- ensemblGenome()
 
# read GTF file into ensemblGenome object
read.gtf(ens, "Mus_musculus.GRCm38.91.gtf")

[read.gtf.refGenome] Reading file 'Mus_musculus.GRCm38.91.gtf'.
[GTF]  1768665 lines processed.
[read.gtf.refGenome] Extracting genes table.
**[read.gtf.refGenome] Found 53,465 gene records.**
[read.gtf.refGenome] Finished.
```


看看这个ens是个啥

```
  ens
 Object of class 'ensemblGenome' with 1,715,195 rows and 31 columns.
   id seqid  source    feature   start     end score strand frame
 2  2     1  havana transcript 3073253 3074322     .      +     .
 3  3     1  havana       exon 3073253 3074322     .      +     .
 5  5     1 ensembl transcript 3102016 3102125     .      +     .
 6  6     1 ensembl       exon 3102016 3102125     .      +     .
 8  8     1  havana transcript 3205901 3216344     .      -     .
 9  9     1  havana       exon 3213609 3216344     .      -     .
   protein_version protein_id ccds_id transcript_support_level   tag
 2            <NA>       <NA>    <NA>                       NA basic
 3            <NA>       <NA>    <NA>                       NA basic
 5            <NA>       <NA>    <NA>                       NA basic
 6            <NA>       <NA>    <NA>                       NA basic
 8            <NA>       <NA>    <NA>                        1  <NA>
 9            <NA>       <NA>    <NA>                        1  <NA>
     transcript_biotype transcript_source exon_number            gene_id
 2                  TEC            havana        <NA> ENSMUSG00000102693
 3                  TEC            havana           1 ENSMUSG00000102693
 5                snRNA           ensembl        <NA> ENSMUSG00000064842
 6                snRNA           ensembl           1 ENSMUSG00000064842
 8 processed_transcript            havana        <NA> ENSMUSG00000051951
 9 processed_transcript            havana           1 ENSMUSG00000051951
     transcript_name gene_version  havana_transcript     gene_name
 2 4933401J01Rik-201            1 OTTMUST00000127109 4933401J01Rik
 3 4933401J01Rik-201            1 OTTMUST00000127109 4933401J01Rik
 5       Gm26206-201            1               <NA>       Gm26206
 6       Gm26206-201            1               <NA>       Gm26206
 8          Xkr4-203            5 OTTMUST00000086625          Xkr4
 9          Xkr4-203            5 OTTMUST00000086625          Xkr4
      gene_source   gene_biotype exon_version        havana_gene
 2         havana            TEC         <NA> OTTMUSG00000049935
 3         havana            TEC            1 OTTMUSG00000049935
 5        ensembl          snRNA         <NA>               <NA>
 6        ensembl          snRNA            1               <NA>
 8 ensembl_havana protein_coding         <NA> OTTMUSG00000026353
 9 ensembl_havana protein_coding            1 OTTMUSG00000026353
   havana_transcript_version havana_gene_version            exon_id
 2                         1                   1               <NA>
 3                         1                   1 ENSMUSE00001343744
 5                      <NA>                <NA>               <NA>
 6                      <NA>                <NA> ENSMUSE00000522066
 8                         1                   2               <NA>
 9                         1                   2 ENSMUSE00000858910
   transcript_version      transcript_id
 2                  1 ENSMUST00000193812
 3                  1 ENSMUST00000193812
 5                  1 ENSMUST00000082908
 6                  1 ENSMUST00000082908
 8                  1 ENSMUST00000162897
 9                  1 ENSMUST00000162897


class(ens)
[1] "ensemblGenome"

```

emmm,和之前介绍过的格式差不多，请自行理解。


### 2. 功能简介

#### genome class相关操作

##### 1. 通过染色体相关信息（Seqids） 提取ens子集，对象类型是ens对应的ensemblGenome
 - `extractSeqids`：提取特定染色体上的基因注释信息
 - `tableSeqids`：统计特定染色体上的基因

比如取出线粒体染色体上的信息：`extractSeqids(ens, "^MT$")`, MT即线粒体的代号，这里用了正则表达式（^和$）做了过滤。
那怎么看可以用的染色体信息呢：输入`ensPrimAssembly()`，可以得到 primary assembly data，用于正则表达式筛选
```
[1] "^([0-9]{1,2})$|^[XY]|MT$"
```

```
enpa <- extractSeqids(ens,ensPrimAssembly())
tableSeqids(enpa)

     1     10     11     12     13     14     15     16     17     18     19 
111746  83081 123843  60654  62035  70962  57035  48515  68654  31782  45802 
     2      3      4      5      6      7      8      9     MT      X      Y 
146388  83712 100869 118010  94207 146202  82914 105329    110  60507  11953 
```

假如不加任何过滤信息，可以得到正常ensembl里没有的特异信息，可能是新的参考基因组里出现的：

```
 tableSeqids(ens)

#         1         10         11         12         13         14         15 
    111746      83081     123843      60654      62035      70962      57035 
#        16         17         18         19          2          3          4 
     48515      68654      31782      45802     146388      83712     100869 
         5          6          7          8          9 GL456210.1 GL456211.1 
#    118010      94207     146202      82914     105329         37         50 
GL456212.1 GL456216.1 GL456219.1 GL456221.1 GL456233.1 GL456239.1 GL456350.1 
#        34         21         10         56         61          2         60 
GL456354.1 GL456372.1 GL456381.1 GL456385.1 JH584292.1 JH584293.1 JH584294.1 
#        48          2          2          4         75        100         83 
JH584295.1 JH584296.1 JH584297.1 JH584298.1 JH584299.1 JH584303.1 JH584304.1 
#        15         25         25         29        110          4         32 
        MT          X          Y 
#       110      60507      11953 
```


##### 2. 通过features提取ens子集 ，对象还是ensemblGenome
 -  `extractFeature`
 - `tableFeatures`

```
enpf <- extractFeature(enpa,"exon")
enpf

Object of class 'ensemblGenome' with 794,728 rows and 31 columns.
   id seqid  source feature   start     end score strand frame protein_version
3   3     1  havana    exon 3073253 3074322     .      +     .            <NA>
6   6     1 ensembl    exon 3102016 3102125     .      +     .            <NA>
10 10     1  havana    exon 3205901 3207317     .      -     .            <NA>

tableFeatures(enpa)

            CDS            exon  five_prime_utr  Selenocysteine     start_codon 
         504216          794728           90176              65           57074 
     stop_codon three_prime_utr      transcript 
          53106           81096          133849 

```

##### 3. 提取单个基因或者转录本信息，举在上文中提到的Xkr4 基因（ENSMUSG00000051951，哈哈）

`extractByGeneId(ens, "ENSMUSG00000051951")`
>![](http://owxb9z5ea.bkt.clouddn.com/18-3-30/32797111.jpg)

` extractTranscript(ens, "ENSMUST00000162897")`
>Object of class 'ensemblGenome' with 3 rows and 31 columns.
>![](http://owxb9z5ea.bkt.clouddn.com/18-3-30/72540956.jpg)

还可以用`tableTranscript.id`看看所有转录本的信息

```
> head(tableTranscript.id(ens))

ENSMUST00000000001 ENSMUST00000000003 ENSMUST00000000010 ENSMUST00000000028 
                23                 19                  9                 45 
ENSMUST00000000033 ENSMUST00000000049 
                13                 21 
```


##### 5. 信息整合
比如这个任务：Accumulate data for whole genes
原文描述比较精妙：
> The function getGenePositions accumulates position data for whole genes. Genes are grouped by gene_name. For both, ensemblGenome and ucscGenome the gene_name column is not present after the standard gtf-import. For ucscGenome, addXref must be used. Respective warnings are thrown.

```
gpe <- getGenePositions(ens)
gpe
            id      seqid         source   start     end score strand frame
    1:       1          1         havana 3073253 3074322     .      +     .
    2:       4          1        ensembl 3102016 3102125     .      +     .
    3:       7          1 ensembl_havana 3205901 3671498     .      -     .
    4:      25          1         havana 3252757 3253236     .      +     .
    5:      28          1         havana 3365731 3368549     .      -     .
   ---                                                                     
53461: 1768560 GL456385.1        ensembl   32719   32818     .      +     .
53462: 1768563 GL456372.1        ensembl   13262   13382     .      -     .
53463: 1768566 GL456381.1        ensembl   16623   16721     .      -     .
53464: 1768569 JH584292.1        ensembl    3536   11935     .      +     .
53465: 1768645 JH584295.1        ensembl      66    1479     .      -     .
```


#### Exon class相关操作：仅适用于ensembl data
splice-junction的用法从略，感觉不是这个包干的主要任务，在有兴趣可自行探索。:)
`ensemblExons`
`getSpliceTable(ens)` : splice-junction based views

#### geneModel和transcriptModel对象相关操作

##### 1. geneModel对象
包含若干转录本对象和外显子对象。
 - `getGeneTable`
 - `geneModel`

```
gt <- getGeneTable(ens)
> head(gt)
   id seqid         source   start     end score strand frame protein_version
1   1     1         havana 3073253 3074322     .      +     .            <NA>
4   4     1        ensembl 3102016 3102125     .      +     .            <NA>
7   7     1 ensembl_havana 3205901 3671498     .      -     .            <NA>
25 25     1         havana 3252757 3253236     .      +     .            <NA>
28 28     1         havana 3365731 3368549     .      -     .            <NA>
31 31     1         havana 3375556 3377788     .      -     .            <NA>
gm <- geneModel(ens, "ENSMUSG00000051951")
plot(gm)1
```

>![](http://owxb9z5ea.bkt.clouddn.com/18-3-30/1397243.jpg)


##### 2. transcriptModel对象
主要包含单个转录本信息，比如geneid, genename, strand or coordinates. 可以通过name或这index从geneModel objects 提取这类对象。
`getTranscript`
`getExonData`


__Extraction by index__
```
> getTranscript(gm, 1)
An object of class 'transcriptModel'.
ID          :  ENSMUST00000070533 
Name        :  Xkr4-201 
Gene ID     :  ENSMUSG00000051951 
Gene Name   :  Xkr4 
Start       :  3,214,482    End         :  3,671,498 
Start codon :  3,671,346    Stop  codon :  3,216,022 
5' prime utr:  3,671,349    3' prime utr:  3,214,482 
Seq Name    :  1 
Strand      :  - 
```

__Extraction by ID__
```
tr <- getTranscript(gm, "ENSMUST00000070533")
tr

An object of class 'transcriptModel'.
ID          :  ENSMUST00000070533 
Name        :  Xkr4-201 
Gene ID     :  ENSMUSG00000051951 
Gene Name   :  Xkr4 
Start       :  3,214,482    End         :  3,671,498 
Start codon :  3,671,346    Stop  codon :  3,216,022 
5' prime utr:  3,671,349    3' prime utr:  3,214,482 
Seq Name    :  1 
Strand      :  - 
```

其他
```
getExonData(tr)
    start     end            exon_id exon_number exon_version seqid strand
1 3670552 3671498 ENSMUSE00000485541           1            3     1      -
2 3421702 3421901 ENSMUSE00000449517           2            3     1      -
3 3214482 3216968 ENSMUSE00000448840           3            2     1      -


getCdsData(tr)
    start     end            exon_id exon_number exon_version seqid strand
1 3670552 3671498 ENSMUSE00000485541           1            3     1      -
2 3421702 3421901 ENSMUSE00000449517           2            3     1      -
3 3214482 3216968 ENSMUSE00000448840           3            2     1      -

```


#### 最后：一些简单的结合分析

```
# length of genes
my_gene_length <- log10(gt$end - gt$start)
my_density <- density(my_gene_length)
plot(my_density, main = 'Distribution of gene lengths (log10, bp)')
```

![](http://owxb9z5ea.bkt.clouddn.com/18-3-30/23231531.jpg)


```
library(GenomicRanges)
my_gr <- with(my_gene, GRanges(seqid, IRanges(start, end), strand, id = gene_id))
 
library(Gviz)
ref <- GRanges()
ref_track <- GenomeAxisTrack()
options(ucscChromosomeNames=FALSE)
data_track <- AnnotationTrack(my_gr, name = "Genes", showFeatureId = TRUE)
plotTracks(c(ref_track, data_track),
           from = 1, to = 10000)## 图略略略略。。。
```


> REF:
1. https://cran.r-project.org/web/packages/refGenome/vignettes/refGenome.pdf
2. https://davetang.org/muse/2017/08/04/read-gtf-file-r/
