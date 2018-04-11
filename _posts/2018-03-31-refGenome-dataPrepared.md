---
layout: post
title: 'genome browser(IGV为例) 上文件的准备'
category: Analysis 
tags: ['genome browser','data format']
excerpt_separator: "<!--more-->"
created: 2018-03-31T12:50:31-15:50:00
last_modified_at: 2018-04-11T12:25:31-12:50:00
---

Q: 何快速有效地将新拿到的参考基因组在IGV里可视化？

# 举一个栗子
 - IGV：安装就不讲了，自行百度
 - 物种：schizosaccharomyces_pombe
 - 必需：FA文件，纯序列信息，分染色体记录
 - 基因信息：refGene.txt 需要对应FA参考基因组版本，通常在一个目录下就能获得，没有的时候需要用gtfToGenePred将gtf信息转换成对应格式。

FA文件请自行到ftp://ftp.ensemblgenomes.org/pub/fungi/current/  寻找对应的物种，下面主要讲注释信息格式的转换：

# Data preparation 

 1. 序列的GTF文件
了解[gtf格式](http://genomewiki.ucsc.edu/index.php/Genes_in_gtf_or_gff_format)

`wget ftp://ftp.ensemblgenomes.org/pub/fungi/current/gtf/schizosaccharomyces_pombe/Schizosaccharomyces_pombe.ASM294v2.38.gtf`
_#注：版本号不同可能会没有这个文件，回到上层目录即可下载_


 2. 将GTF转化成genePred，再提取出refGene.txt 
 - GTF 文件
wget ftp://ftp.ensemblgenomes.org/pub/fungi/current/gtf/schizosaccharomyces_pombe/Schizosaccharomyces_pombe.ASM294v2.38.gtf
 - refGene也属于[GenePred table format](http://genome.ucsc.edu/FAQ/FAQformat):

```
table genePredExt
"A gene prediction with some additional info."
    (
    string name;        	"Name of gene (usually transcript_id from GTF)"
    string chrom;       	"Chromosome name"
    char[1] strand;     	"+ or - for strand"
    uint txStart;       	"Transcription start position"
    uint txEnd;         	"Transcription end position"
    uint cdsStart;      	"Coding region start"
    uint cdsEnd;        	"Coding region end"
    uint exonCount;     	"Number of exons"
    uint[exonCount] exonStarts; "Exon start positions"
    uint[exonCount] exonEnds;   "Exon end positions"
    int score;            	"Score"
    string name2;       	"Alternate name (e.g. gene_id from GTF)"
    string cdsStartStat; 	"enum('none','unk','incmpl','cmpl')"
    string cdsEndStat;   	"enum('none','unk','incmpl','cmpl')"
    lstring exonFrames; 	"Exon frame offsets {0,1,2}"
    )
```

# 理想化的流程和遇到的问题

思路如下：
1. 根据IGV的对注释文件的需求：http://software.broadinstitute.org/software/igv/genePred
我们需要用[gtfToGenePred]( http://hgdownload.soe.ucsc.edu/admin/exe/) 将下载到的gtf转换成通用的genePred格式。格式见文末。

>GenePred is a table format commonly used for gene prediction tracks in the UCSC Genome Browser. The genePredToGtf command-line utility can be used to convert genePred to GTF.

2.  但是获得的genepred直接导入igv不成功：

![](http://owxb9z5ea.bkt.clouddn.com/18-3-31/94419347.jpg)

比较了一下正常能够显示序列信息的refgene.txt格式发现相比genepred文件第一列多了数字，
那来源是什么？

refGene的格式,比如小鼠的 http://hgdownload.soe.ucsc.edu/goldenPath/mm10/database/refGene.txt

```
head /nethome/Shared/annotation/iGenome/Mus_musculus/UCSC/mm9/Annotation/Genes/refGene.txt 
1669    NM_001083312    chr3    +    142193301    142213047    142197486    142209538    11    142193301,142197480,142199260,142200975,142201685,142204319,142205812,142206885,142208413,142208825,142209283,    142193403,142197676,142199388,142201085,142201882,142204565,142206093,142207098,142208516,142209019,142213047,    0    Gbp7    cmpl    cmpl    -1,0,1,0,2,1,1,0,0,1,0,
1348    NM_001160108    chr12    +    100009231    100025984    100009465    100024791    7    100009231,100012454,100021791,100023125,100023527,100023844,100024784,100009859,100012529,100021912,100023262,100023616,100023951,100025984,    0    Zc3h14    cmpl    cmpl    0,1,1,2,1,0,2,
1465    NR_029786    chr14    +    115443221    115443303    115443303    115443303    1    115443221,    115443303,    0    Mir19a    unk    unk    -1,
1465    NR_029785    chr14    +    115442892    115442976    115442976    115442976    1    115442892,    115442976,    0    Mir17    unk    unk    -1,
667    NR_029781    chr18    -    10785478    10785550    10785550    10785550    1    10785478,    10785550,    0    Mir1a-2    unk    unk    -1,
1514    NM_001115074    chr1    -    121820095    121891787    121820095    121891787    23    121820095,121822137,121824190,121827465,121829350,121830658,121831097,121832667,121833818,121842359,121844628,121846247,121847626,121850177,121850478,121850724,121852225,121858448,121878290,121881306,121881592,121886006,121891648,    121820204,121822233,121824283,121827557,121829509,121830790,121831205,121832748,121833946,121842465,121844727,121846337,121847714,121850286,121850590,121850842,121852385,121858552,121878391,121881405,121881679,121886107,121891787,    0    Gm101    cmpl    cmpl    2,2,2,0,0,0,0,0,1,0,0,0,2,1,0,2,1,2,0,0,0,1,0,
```

但是GenePred文件第一列没有这个id

```
 ==> gene_info-Schizosaccharomyces_pombe.ASM294v2.38.txt <==
SPAC212.11.1	I	-	0	5662	0	5662	1	0,	5662,	0	SPAC212.11	incmpl	cmpl	0,
SPAC212.10.1	I	-	5725	6331	6331	6331	1	5725,	6331,	0	SPAC212.10	none	none	-1,
SPAC212.09c.1	I	+	7618	9274	9274	9274	1	7618,	9274,	0	SPAC212.09c	none	none	-1,
SPNCRNA.70.1	I	-	11026	11556	11556	11556	1	11026,	11556,	0	SPNCRNA.70	none	none	-1,
SPAC212.08c.1	I	+	11783	12994	12157	12994	1	11783,	12994,	0	SPAC212.08c	cmpl	cmpl	0,
```


经查阅genePredExt第十二列的信息为refgene.txt的第一列，所以只要加把第十二列提到第一列即可：
 string name2; "Alternate name (e.g. gene_id from GTF)"

```
> genePredExt格式
"A gene prediction with some additional info."
    (
    string name; "Name of gene (usually transcript_id from GTF)"
    string chrom; "Chromosome name"
    char[1] strand; "+ or - for strand"
    uint txStart; "Transcription start position"
    uint txEnd; "Transcription end position"
    uint cdsStart; "Coding region start"
    uint cdsEnd; "Coding region end"
    uint exonCount; "Number of exons"
    uint[exonCount] exonStarts; "Exon start positions"
    uint[exonCount] exonEnds; "Exon end positions"
    int score; "Score"
    __string name2; "Alternate name (e.g. gene_id from GTF)"__
    string cdsStartStat; "enum('none','unk','incmpl','cmpl')"
    string cdsEndStat; "enum('none','unk','incmpl','cmpl')"
    lstring exonFrames; "Exon frame offsets {0,1,2}"
    )
```


# 最终流程
参考 http://genomewiki.ucsc.edu/index.php/Genes_in_gtf_or_gff_format#Bed_format_gene_tracks_.28convert_bed_.3E_genePred_.3E_GTF.29

大神版
```
gtfToGenePred -genePredExt -geneNameAsName2 Schizosaccharomyces_pombe.ASM294v2.38.gtf refFlat.tmp.txt
paste <(cut -f 12 refFlat.tmp.txt) <(cut -f 1-10 refFlat.tmp.txt) > refFlat.txt
```

OR 小白版:
```
1. Download http://hgdownload.cse.ucsc.edu/admin/exe/linux.x86_64/gtfToGenePred.
2. Convert a downlaoded gtf annotation file into genepred extended format (tmp file)
./gtfToGenePred -genePredExt annotation.gtf tmp
3. Parse the tmp file into genepred format 
awk 'BEGIN{FS="\t"};{print $12"\t"$1"\t"$2"\t"$3"\t"$4"\t"$5"\t"$6"\t"$7"\t"$8"\t"$9"\t"$10}' tmp > annotation.refFlat
```


#_额外buff_
 1. 可以使用samtools faidx列一下index信息获得fai文件，省得在igv里做：`samtools faidx fasta/schsac.pombe.fa `
 2. 顺道从fai提取chromsize，方便bw文件格式转换：`cat fasta/schsac.pombe.fa.fai | cut -f 1,2 > schsac.pombe.chrom.sizes`


>REF

https://www.biostars.org/p/18480/

