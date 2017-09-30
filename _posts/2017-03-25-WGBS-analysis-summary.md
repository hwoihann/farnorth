---

layout: post
title: "WGBS analysis summary"
categories: definition
excerpt_seperator: <!--more-->
tags: [bismark, methpipe, WGBS]

---
# 前言
年初接触了WGBS数据分析，粗浅总结一下，有待修补:full_moon_with_face:。WGBS数据是精确到每个位点的深度测序，数据量因此也很大，动辄就上百个G，但是分析起来却很有意思。有了各种genomic features就可以探索很多事。
<!--more-->

# 基本概念
__Methylation levels：位点甲基化水平的无偏估计__
Although individual methyl groups can either be present or absent on a cytosine, current WGBS data is not from single cells, and we therefore refer to a methylation ‘‘level’’ for each cytosine, interpreted as the fraction of molecules in the underlying cell population that have the methyl mark on the corresponding cytosine.
Since bisulfite sequencing gives a readout for either the presence or absence of a methyl group in each read, we get an unbiased estimate for the methylation level from the ratio of methylated reads to all reads covering a given site. The estimated cytosine methylation level, along with the number of sequenced reads supporting that estimate, is computed for each methylome, along with c


Visual examination of the methylation patterns at specific genes and genomic loci is an essential part of exploratory data analysis for investigators using WGBS data. 
The duplicate-removal procedure is done on a per-library basis, as any reads from different libraries are necessarily from distinct molecules. W 


## mapping原理及相关软件
read mapping program for bisulfite sequencing in DNA methylation studies.
### 1. bismark：比四次找最优map位置，在map位置上记录甲基化信息
https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3102221/

得到Reads后进行序列比对，从Bisulfite处理到比对过程如下图所示：




在上图A中，一条DNA片段（genomic fragment）经过Bisulfite处理后未甲基化的C转化成T（sequence after bisulfite treatment），将该条Reads比对到基因组上，首先将该Reads进行C to T和G to A转换，再将基因组的模板链进行C to T和G to A转换，将转换后的Reads与转换后的基因组进行比对，在图A中可以看到，（1）为最好的唯一比对结果。


那么如何来判断C位点的甲基化情况呢？
将Reads与模板链还原，如图B所示Reads与模板链的对应关系，如果是C与T对应，则该位点是一个为未甲基化C，如果是C与C对应，则该位点是一个甲基化C。





### 2. methpipe: WALT
WALT (Wildcard ALignment Tool)
https://github.com/smithlabcode/walt
曾经用： rmapbs program, one of the wildcard based mappers.
Input reads are filtered by their quality, and adapter sequences in the 3′ end of reads are trimmed before mapping. Next, uniquely mapped reads with mismatches below a certain threshold are kept for further analysis. For pair-end reads, two mates from the same original DNA segment are mapped simultaneously. The rmapbs-pe program checks the appropriate configuration of the mate pairs. If the two mates overlap, the overlapping part of the mate with lower quality is clipped. This procedure prevents double counting. Users may also use alternative mappers (see [14] and [15] for reviews). Our pipeline provides a program that conveniently converts the output of alternative mappers to the format supported by MethPipe.

## 实例：
PE情况下：read1: C -> T read2: G -> A（实验过程中已完成） 
下图分别为 H:\allProject\Project_scnt\WGBS_new\3M\0_trim_se\auto 中

SCNT1_1， SCNT2_1的fastqc报告结果
可以看到read1 T含量很高， read2 A含量很高

 
 



#流程
##1. 总览
DNA methylome analysis using short bisulfite sequencing data 必须入门
http://www.nature.com/nmeth/journal/v9/n2/full/nmeth.1828.html

 

## 2. QC

__trimGalore__
官方文档：
https://github.com/FelixKrueger/TrimGalore/blob/master/Docs/Trim_Galore_User_Guide.md

> 引自methpipe
__误差来源__
Empirical benchmarks for new WGBS experiments
Technical characteristics of publicly available WGBS methylomes vary considerably.
 - Part of this results from the decisions made by investigators concerning parameters such as read length and sequencing depth.  
 - Additional variation comes from poor experimental outcomes:
reflected in low bisulfite conversion, or read coverage that is biased towards or away from CpG-dense regions. This variation raises the question of how to evaluate the quality of methylomes from WGBS or RRBS experiments. The large and growing collection of methylomes in MethBase provides an empirical benchmark for future investigations. 
 - One potentially important feature that varies perhaps unexpectedly is the relationship between CpG density and coverage depth in WGBS data. Using the methylation data from a subset of human WGBS methylomes, we computed the average coverage at CpG sites and the CpG density in each 1 kb bin through the human autosomes, which surprisingly shows significant correlation in a large number of samples (Figure 4B and Table S1). The correlation may be either positive or negative, even for samples from the same study (Table S1). It remains unclear whether such correlation reflects technical artifacts or any underlying biological phenomena. However, methylation features have several well-known relationships with CpG density, and our confidence in methylation level estimates is a function of depth of coverage. The unexpected correlation between read coverage and CpG density may affect our ability to assess the methylation status of regions with varying CpG density.

## 3. mapping & feature analysis 
主要就是找差异甲基化，在感兴趣的区域上看甲基化的情况呗～
### 3.1 methPipe
http://smithlabresearch.org/software/methpipe/

> 一些用到的算法大致介绍
 - __Identifying Hypo-methylated Regions__
 methylation level and coverage at individual cytosines along the genome
 two-state HMM-based method:
Introduces one state representing hypo-methylated regions, and another state representing highly methylated background. Since the estimated methylation levels suffer large variance due to randomly sampled molecules (especially at lower coverage), we directly model the observed read counts with a __beta-binomial distribution__, for which we implemented a rapid and numerically stable parameter estimation method. The HMM is trained using the Baum-Welch algorithm [38], and posterior decoding identifies HMRs. The method is implemented in the hmr program. Median HMR size and the number of HMRs directly result from this computation, and can provide a quick and useful characterization of a mammalian methylome.

 - __Identifying Hyper-methylated Regions__
HyperMRs are also inferred from the methylation level and coverage at individual cytosines, with a __stochastic segmentation model__ similar to the HMR-finding method. Visual examination of methylation patterns of multiple samples motivated us to model the methylation status of cytosines in the Arabidopsis genome with three states: the hypo-methylated state in the background, the HYPER-methylated state in HyperMRs and the HYPO-methylated state scattered within HyperMRs. In addition, the length of genomic regions is modeled with explicit duration HMM (“variable duration” HMM; [39]). Explicit duration distribution generalizes the implicit geometric duration distribution in HMM, and is more flexible. Detailed mathematical formulation is given in supplementary material. The method is implemented in hmr_plant program.

 - __Identifying Partially Methylated Domains__
As explained above, PMDs are large-scale methylation features found in immortalized cell lines and cancerous cells. We developed a hierarchical method to identify PMDs. First, the genome is divided into 1 kb non-overlapping bins (an empirically determined value that remains user-adjustable). We count the number of methylated and unmethylated observations (i.e. CpG states in reads) in each bin. Similar to the HMR-finding method, we use a two-state HMM model to segment the genome into PMDs and background regions. This step at low resolution gives the location of PMDs, while the exact location of their boundaries need further refinement. Since the boundary between a PMD and a non-PMD region must reside within the two bins at the end of that PMD and that non-PMD region, the refinement of PMD boundaries reduces to detecting a single change-point along the sequence of sites in those two bins. The single change-point detection problem has been extensively studied, and we used the binary segmentation procedure of Sen et al. [40].

 - __Identifying Allele-specific Methylated Regions__
To identify allele-specific methylated regions, we use the linkage information of the methylation status between adjacent cytosines in a read. The separation of reads into two alleles and the testing of whether a certain region fits the allele-specific model is carried out with the statistical method described by Fang et al. [17]. Additionally, a single-site profile for an allele-specific methylation “score” can be computed along the genome by testing for significance of linkage between methylation status in reads covering adjacent CpGs. The programs for identifying AMRs and computing allelic scores are amrfinder and allelicmeth.

 - __Identifying Differentially Methylated Regions__
Comparative studies of methylomes usually involves identifying DMRs between samples from different conditions. MethPipe includes two different methods to identify DMRs between two methylomes, each with its applicable situations. The first method extends that introduced by [10]. Differential methylation scores are first computed at individual CpG sites, taking into account observed frequencies of methylation as well as the amount of data contributing to those frequencies [41]. Next, non-overlapping parts of HMRs (i.e. the contiguous parts of the symmetric difference of the two interval sets) are evaluated to ensure they contain a sufficient number of differentially methylated CpG sites. This strategy is HMR-centric, and makes sure that the absense of an HMR in one methylome is not simply due to lack of data. This is implemented in the program dmr. The second method uses a three-state HMM to segment the methylome into contiguous sets of CpGs that are either (i) not differentially methylated, (ii) have an over-abundance of significantly different CpG sites in one direction, and (iii) have an over-abundance in the other direction (details in supplementary material). The second method is sensitive to DMRs caused by partial methylation variation, and is implemented in the dmr2 program.

 - __Cross-species Comparison of Methylomes__
To facilitate cross-species comparison of methylomes of multiple species, we converted the methylation data of mouse and chimpanzee to the corresponding locations in the human genome. The liftOver tool provided by UCSC Genome Browser http://genome.ucsc.edu/cgi-bin/hgLiftOver is used to directly convert the methylation level file at individual cytosines (output from the program methcounts ) to the human genome. Next we rerun the hmr and pmd programs on the converted methylation data file. Since the program amr works on the mapped read file, we directly liftover the list of AMRs with the liftOver program.

---

__实例__

 1. 数据准备，用bismark的mapping方法，获得的bam 用to-mr转换成methpipe可以处理的格式
```
#!!+===========================================================
# mapping using BISMARK, extracting using methpipe:
##============================================================
## merge old data
samtools merge 2_bismark_se/SCNT_kdm4d.bam 2_oldBis/SCNT_Kdm4d*.bam
samtools merge 2_bismark_se/IVF.bam 2_oldBis/IVF*.bam
## merge new data
samtools merge 3_bismark_SAM/SCNT.bam  2_bismark_pe/SCNT*.bam 2_bismark_se/SCNT3.fq_bismark_bt2.bam  ##-f to overwrite
samtools view -b bismark 2_bismark_se/IVF.bam 
## convert into methpipe format
to-mr -o 4_methPipe/IVF.mr -m bismark 2_bismark_se/IVF.bam 
to-mr -o 4_methPipe/SCNT_kdm4d.mr -m bismark 2_bismark_se/SCNT_kdm4d.bam 
to-mr -o 4_methPipe/SCNT.mr -m bismark 3_bismark_SAM/SCNT.bam
to-mr -o 4_methPipe/SCNT.mr -m bismark 2_bismark_pe/SCNT.bam
```

 2. bsrate, methcounts
```
## remove duplicated reads:: map identical genomic regions
cd 4_methPipe

### methylation conversion & remove duplications
 LC_ALL=C sort -k 1,1 -k 2,2n -k 3,3n -k 6,6 -o IVF.mr.sorted_start IVF.mr
 LC_ALL=C sort -k 1,1 -k 2,2n -k 3,3n -k 6,6 -o SCNT_kdm4d.mr.sorted_start SCNT_kdm4d.mr
LC_ALL=C sort -k 1,1 -k 2,2n -k 3,3n -k 6,6 -o SCNT.mr.sorted_start SCNT.mr

duplicate-remover -S IVF_dremove_stat.txt -o IVF.mr.dremove IVF.mr.sorted_start
duplicate-remover -S SCNT_Kdm4d_dremove_stat.txt -o SCNT_kdm4d.mr.dremove  SCNT_kdm4d.mr.sorted_start
duplicate-remover -S SCNT_dremove_stat.txt -o SCNT.mr.dremove SCNT.mr.sorted_start

## estimate the bisulfite conversion rate ##
bsrate -c  /nethome/Shared/annotation/iGenome/Mus_musculus/UCSC/mm9/Sequence/Bowtie2Index/ -o IVF.bsrate IVF.mr.dremove
bsrate -c  /nethome/Shared/annotation/iGenome/Mus_musculus/UCSC/mm9/Sequence/Bowtie2Index/ -o SCNT_kdm4d.bsrate SCNT_kdm4d.mr.dremove 
bsrate -c  /nethome/Shared/annotation/iGenome/Mus_musculus/UCSC/mm9/Sequence/Bowtie2Index/ -o SCNT.bsrate SCNT.mr.dremove 

### calculate the methylation value of single site of files with dupilicated reads removed
### 
methcounts -c  /nethome/Shared/annotation/iGenome/Mus_musculus/UCSC/mm9/Sequence/Bowtie2Index/  -o IVF.meth IVF.mr.dremove
methcounts -c  /nethome/Shared/annotation/iGenome/Mus_musculus/UCSC/mm9/Sequence/Bowtie2Index/  -o SCNT_kdm4d.meth  SCNT_kdm4d.mr.dremove 
methcounts -c  /nethome/Shared/annotation/iGenome/Mus_musculus/UCSC/mm9/Sequence/Bowtie2Index/  -o SCNT.meth  SCNT.mr.dremove 

#假如只需要CpG信息，不用grep，直接在methcounts 里加参数-n即可，此处用对获得的所有C信息进行grep获得CpG信息
### grep the CpG sites
grep CpG SCNT.meth > SCNT_CpG.meth
grep CpG IVF.meth >  IVF_CpG.meth
grep CpG SCNT_kdm4d.meth > SCNT_kdm4d_CpG.meth

### destranded the data
symmetric-cpgs -o SCNT_CpG.destranded.meth SCNT_CpG.meth
symmetric-cpgs -o IVF_CpG.destranded.meth IVF_CpG.meth
symmetric-cpgs -o SCNT_kdm4d_CpG.destranded.meth SCNT_kdm4d_CpG.meth
```
 3. extract 5x CpG for UCSC visualization

```
## extract 5x CpG sites in SCNT and IVF blastocysts (0-based, destranded, sorted)

##compare with old data
# 20605234	IVF_CpG.meth
# 20670958	SCNT_CpG.meth

#  20605234 4_methPipe/CpG_IVF_150203.meth
#  20670958 4_methPipe/CpG_SCNT_150203.meth

#  20524342 4_methPipe/IVF_CpG.destranded.meth
#  20778281 4_methPipe/SCNT_CpG.destranded.meth
#  20595095 4_methPipe/SCNT_kdm4d_CpG.destranded.meth

## old data format: 
## chr 	start(0) 	+ 	CpG 	ratio 	coverage
## convert to coverage and methylation levels(更通用)
# chr 	start(0)	end(0)	ratio*100 	meth 	unmeth
awk 'BEGIN{FS=OFS="\t"; OFMT="%.2f"} ($6>=5){print $1,$2,$2,$5*100,$6*$5,$6-$6*$5;}' CpG_IVF_150203.meth > ./destranded_5x/IVF_15Feb.bed
awk 'BEGIN{FS=OFS="\t"; OFMT="%.2f"} ($6>=5){print $1,$2,$2,$5*100,$6*$5,$6-$6*$5;}' CpG_SCNT_150203.meth > ./destranded_5x/SCNT_kdm4d_15Feb.bed

awk 'BEGIN{FS=OFS="\t"; OFMT="%.2f"} ($6>=5){print $1,$2,$2,$5*100,$6*$5,$6-$6*$5;}' SCNT_kdm4d_CpG.destranded.meth > ./destranded_5x/SCNT_kdm4d_CpG.bed
awk 'BEGIN{FS=OFS="\t"; OFMT="%.2f"} ($6>=5){print $1,$2,$2,$5*100,$6*$5,$6-$6*$5;}' IVF_CpG.destranded.meth > ./destranded_5x/IVF_CpG.bed
awk 'BEGIN{FS=OFS="\t"; OFMT="%.2f"} ($6>=5){print $1,$2,$2,$5*100,$6*$5,$6-$6*$5;}' SCNT_CpG.destranded.meth > ./destranded_5x/SCNT_CpG.bed

  ## in 4_methPipe/destranded_5x/
  12563096 IVF_15Feb.bed  
  12718231 IVF_CpG.bed
  12355989 SCNT_kdm4d_15Feb.bed
  12508668 SCNT_kdm4d_CpG.bed
  13434019 SCNT_CpG.bed

## make bigWig files (methylation and coverage)
mkdir 5_visual/bdg 5_visual/wig 

cd ./4_methPipe/destranded_5x

for file in *.bed; do
	newfile=${file%%.*}
	echo $newfile
awk 'BEGIN{FS= "\t"; OFS="\t"}; {if($5+$6 >= 5) {print $1, $2-1, $2, $4}}' ${file} > ../../5_visual/bdg/${newfile}_mp_5x.meth.bdg
awk 'BEGIN{FS= "\t"; OFS="\t"}; {if($5+$6 >= 5) {print $1, $2-1, $2, $5+$6}}' ${file} > ../../5_visual/bdg/${newfile}_mp_5x.cov.bdg
wigToBigWig ../../5_visual/bdg/${newfile}_mp_5x.meth.bdg ~/trackTool/mm9.chrom.sizes ../../5_visual/wig/${newfile}_mp_5x.meth.bw
wigToBigWig ../../5_visual/bdg/${newfile}_mp_5x.cov.bdg ~/trackTool/mm9.chrom.sizes ../../5_visual/wig/${newfile}_mp_5x.cov.bw
done

2>&1 | tee ../5_rp_Bismark_bedgraph/rp_extractor_SCNT.txt
然后就上传吧
```
 4. 根据获得的meth文件，汇总甲基化信息 + HMR detection
```
### Generate statistics : useful for comparison
levels -o 4_methPipe/stats/stat_oldIVF_CpG.txt 4_methPipe/CpG_IVF_150203.meth
levels -o 4_methPipe/stats/stat_oldSCNT_kdm4d_CpG.txt 4_methPipe/CpG_SCNT_150203.meth
levels -o 4_methPipe/stats/stat_SCNT_kdm4d_CpG.txt 4_methPipe/SCNT_kdm4d_CpG.meth
levels -o 4_methPipe/stats/stat_IVF_CpG.txt 4_methPipe/IVF_CpG.meth
levels -o 4_methPipe/stats/stat_SCNT_CpG.txt 4_methPipe/SCNT_CpG.meth
```

报告内容和官方说明：
>SITES: 1000000
SITES COVERED: 566157
FRACTION MUTATED: 0.001257
FRACTION COVERED: 0.566157
MAX COVERAGE: 439
SYMMETRICAL CpG COVERAGE: 11.3228
SYMMETRICAL CpG COVERAGE (WHEN > 0): 15.3289
SYMMETRICAL CpG COVERAGE (minus mutations): 11.2842
SYMMETRICAL CpG COVERAGE (WHEN > 0) (minus mutations): 15.2768
MEAN COVERAGE: 3.31458
MEAN COVERAGE (WHEN > 0): 5.85452
METHYLATION LEVELS (CpG CONTEXT):
mean_meth 0.700166
w_mean_meth 0.667227
frac_meth 0.766211
METHYLATION LEVELS (CHH CONTEXT):
mean_meth 0.0275823
w_mean_meth 0.0184198
frac_meth 0.0146346
METHYLATION LEVELS (CXG CONTEXT):
mean_meth 0.0217537
w_mean_meth 0.0170535
frac_meth 0.00843068
METHYLATION LEVELS (CCG CONTEXT):
mean_meth 0.0211243
w_mean_meth 0.0187259
frac_meth 0.00630109
total fraction of cytosines covered, the fraction of cytosines that have mutated away from the reference,  coverage statistics for both CpGs and all cytosines.  
For CpG sites, coverage number reflects taking advantage of their symmetric nature and merging the coverage on both strands. 
For CpG coverage minus mutations, we remove the reads from CpG sites deemed to be mutated away from the reference. 

---

HMR
```
### HMRs Hypo methylated regions ###
hmr -partial -o result/methpipe/hmr/CpG_IVF_150203.hmr 4_methPipe/CpG_IVF_150203.meth
hmr -partial -o result/methpipe/hmr/CpG_SCNT_150203.hmr 4_methPipe/CpG_SCNT_150203.meth

hmr -partial -o result/methpipe/hmr/SCNT_kdm4d_CpG.hmr 4_methPipe/SCNT_kdm4d_CpG.meth
hmr -partial -o result/methpipe/hmr/IVF_CpG.hmr 4_methPipe/IVF_CpG.meth
hmr -partial -o result/methpipe/hmr/SCNT_CpG.hmr 4_methPipe/SCNT_CpG.meth
```
_It also computes average methylation in three different ways, described in Schultz et al. (2012). This program should provide flexibility to compare methylation data with publications that calculate averages different ways and illustrate the variability of the statistic depending on how it is calculated._
 The hmr program uses a hidden Markov model (HMM) approach using a Beta-Binomial distribution to describe methylation levels at individual sites while accounting for the number of reads informing those levels. hmr automatically learns the average methylation levels inside and outside the HMRs, and also the average size of those HMRs(自动计算HMR内外的平均甲基化水平和HMR的平均长度)
Requirements on the data:
We typically like to have about 10x coverage to feel very confident in the HMRs called in mammalian genomes, but the method will work with lower coverage. __The difference is that the boundaries of HMRs will be less accurate at lower coverage, but overall most of the HMRs will probably be in the right places if you have coverage of 5-8x (depending on the methylome).__ Boundaries of these regions are totally ignored by analysis methods based on smoothing or using fixed-width windows.



然后顺便说明训练HMM参数的好处在于甲基化组大部分出现异常的甲基化水平时，基于模型训练得到的HMR更有可比性。可以根据样本的甲基化水平的平均值和方差来设定，方差也受coverage的影响。
-p 训练参数输出
-P 读入训练的参数
疑问：一个文件对应一个训练参数？那么训练参数可以适用一个实验里的所有meth结果吗？

The output will be in BED format, and the indicated strand (always positive) is not informative. The name column in the output will just assign a unique name to each HMR, and the score column indicates how many CpGs exist inside the HMR. Each time the hmr is run it requires parameters for the HMM to use in identifying the HMRs. __We usually train these HMM parameters on the data being analyzed, since the parameters depend on the average methylation level and variance of methylation level__; the variance observed can also depend on the coverage. However, in some cases it might be desirable to use the parameters trained on one data set to find HMRs in another. __The option -p indicates a file in which the trained parameters are written, and the argument -P indicates a file containing parameters (as produced with the -p option on a previous run) to use__: 

```
$ hmr -p Human_ESC.hmr.params -o Human_ESC.hmr Human_ESC.meth
$ hmr -P Human_ESC.hmr.params -o Human_NHFF_ESC_params.hmr Human_NHFF.meth
```

In the above example, the parameters were trained on the ESC methylome, stored in the file Human ESC.hmr.params and then used to find HMRs in the NHFF methylome.  This is useful if a particular methylome seems to have very strange methylation levels through much of the genome, and the HMRs would be more comparable with those from some other methylome if the model were not trained on that strange methylome. 

另有PMR和PMD:
__hmr -partial__
> Partially methylated regions (PMRs):
The hmr program also has the option of directly identifying partially methylated regions (PMRs), not to be confused with partially methylated domains (see below). __These are contiguous intervals where the methylation level at individual sites is close to 0.5.__ This should also not be confused with regions that
have allele-specific methylation (ASM) or regions with alternating high and low methylation levels at nearby sites. Regions with ASM are almost always among the PMRs, but most PMRs are not regions of ASM. The hmr program is run with the same input but a different optional argument to find PMRs: 
```
$ hmr -partial -o Human_ESC.pmr Human_ESC.meth
```

__Partial methylation in cancer samples (AKA PMDs)__
Huge genomic blocks with abnormal hypomethylation have been extensively observed in human cancer methylomes and more recently in extraembryonic tissues like the placenta. These domains are characterized by enrichment in intergenic regions or Lamina associated domains (LAD), which are usually hypermethylated in normal tissues. Partially methylated domains (PMDs) are not homogeneously hypomethylated as in the case of HMRs, and contain focal hypermethylation at specific sites. PMDs are large domains with sizes ranging from 10 kb to over 1 Mb. Hidden Markov Models can also identify these larger domains. The program pmd is provided for their identification, and can be run as follows:
```
$ pmd -o Human_ESC.pmd Human_ESC.meth
```



 - 5. Differential Methylation
 DMR detection 
两种算法：
1. 适用于DMR regions between a pair or two samll groups of methylomes (methdiff & dmr)
2. beta-binomial regression: 适用于样本多的数据集？（radmeth）

```
### DMRs ###
methdiff -o result/dmr/CpG_IVF_SCNT_150202.methdiff methpipe/CpG_IVF_150203.meth methpipe/CpG_SCNT_150203.meth

dmr result/dmr/CpG_IVF_SCNT_150202.methdiff result/hmr/CpG_IVF_150203.hmr methpipe/CpG_SCNT_150203.hmr methpipe/DMR_IVF_high_150203.dmr methpipe/DMR_SCNT_high_150203.dmr
```




```
### Computing the average methylation level in a genomic interval ###
LC_ALL=C sort -k 1,1 -k 3,3n -k 2,2n -k 6,6 -o methpipe/CpG_IVF_150203.meth.sorted methpipe/CpG_IVF_150203.meth
LC_ALL=C sort -k 1,1 -k 3,3n -k 2,2n -k 6,6 -o methpipe/CpG_SCNT_150203.meth.sorted methpipe/CpG_SCNT_150203.meth

roimethstat -P -o methpipe/DMR_IVF_high_IVFmethlevel_150203.txt methpipe/DMR_IVF_high_150203.dmr methpipe/CpG_IVF_150203.meth.sorted
roimethstat -P -o methpipe/DMR_IVF_high_SCNTmethlevel_150203.txt methpipe/DMR_IVF_high_150203.dmr methpipe/CpG_SCNT_150203.meth.sorted
roimethstat -P -o methpipe/DMR_SCNT_high_IVFmethlevel_150203.txt methpipe/DMR_SCNT_high_150203.dmr methpipe/CpG_IVF_150203.meth.sorted
roimethstat -P -o methpipe/DMR_SCNT_high_SCNTmethlevel_150203.txt methpipe/DMR_SCNT_high_150203.dmr methpipe/CpG_SCNT_150203.meth.sorted
```


### 3.2 bismark 
用法简介
https://rawgit.com/FelixKrueger/Bismark/master/Docs/Bismark_User_Guide.html#bismark-alignment-and-methylation-call-report

项目实例：~/Project_scnt/WGBS_new
description: 送去公司测序的样本reads后部分太长质量差需要先经过cut， 再用trimgalore优化剪切reads以达到较好的mapping效果

```
#trim_galore  切除read2 的3'end质量差的base:--three_prime_clip_R1 100 从3'end开始切100bp
trim_galore SCNT1_2.fq -o SE/0_trim --fastqc -o 1_fastqc/se_trim -q 30 --three_prime_clip_R1 100
```

read2 不使用PE而用SE时，bismark设置参数 --non_directional， 否则mapping不了
>The sequencing library was constructed in a non strand-specific manner, alignments to all four bisulfite strands will be reported. 
(The current Illumina protocol for BS-Seq is directional, in which case the strands complementary to the original strands are merely theoretical and should not exist in reality. Specifying directional alignments (which is the default) will only run 2 alignment threads to 
the original top (OT) or bottom (OB) strands in parallel and report these alignments. This is the recommended option for strand-specific libraries). Default: OFF
来源： https://github.com/FelixKrueger/Bismark/tree/master/Docs

```
bismark --bowtie2 -N 1 -p 4 --non_directional /nethome/Shared/annotation/genomes/mm9/Bismark_bowtie2index SE/0_trim/SCNT1_2_trimmed.fq -o ./2_bismark/se_trim 
```

Bismark methylation extractor
Bismark comes with a supplementary bismark_methylation_extractor script which operates on Bismark result files and extracts the methylation call for every single C analysed.
The position of every single C will be written out to a new output file, depending on its context (CpG, CHG or CHH), whereby methylated Cs will be labelled as forward reads (+), non-methylated Cs as reverse reads (-). The resulting files can be imported into a genome viewer such as SeqMonk (using the generic text import filter) and the analysis of methylation data can commence. 

Alternatively, the output of the methylation extractor can be transformed into a bedGraph and coverage file using the option --bedGraph (see also --counts). This step can also be accomplished from the methylation extractor output using the stand-alone script bismark2bedGraph (also part of the Bismark package available for download at bioinformatics.babraham.ac.uk). 
The coverage file can also be imported into SeqMonk directly using Import Data > Bismark (cov). Optionally, the bedGraph counts output can be used to generate a genome-wide cytosine report which reports the number on every single CpG (optionally every single cytosine) in the genome, irrespective of whether it was covered by any reads or not. As this type of report is informative for cytosines on both strands the output may be fairly large (~46mn CpG positions or >1.2bn total cytosine positions in the human genome...). The bedGraph to genome-wide cytosine report conversion can also be run individually using the stand- alone module coverage2cytosine (also part of the Bismark package available for download at bioinformatics.babraham.ac.uk).

bismark report
https://github.com/FelixKrueger/Bismark/blob/master/bismark2report


#意义
获得了unbiased methylome信息，该怎么用呢？首先当然是看一些综述和算法面讲的，下面为一些摘录和理解

[methpipe](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0081148)
>These experiments provide information for numerous regions of dynamic methylation, believed to related with promoters, enhancers, insulators and more broadly regions dense in transcription factor binding sites [11]. Comparative analysis of the methylomes of different cell types and conditions reveals functional epigenetic domains with implications in cell differentiation and disease onset [10], [12], [13].
The __analyses and interpretation__ of bisulfite sequencing data are usually performed in a multi-stage, multi-resolution manner. Extensive efforts have been devoted to mapping reads and __estimating methylation levels at individual cytosine sites__ [14], [15]. Meanwhile, growing interest is shifting towards biologically meaningful higher-level methylation features, such as hypo-methylated regions (HMRs; [3], [9]), large-scale partially methylated domains (PMDs; [7], [13]), and allele-specific methylated regions (AMRs; [16], [17]). Existing methods to identify these are usually project specific, with few general tools available for down-stream analysis tasks.
_Methylation 的计算_
对methylation level很好的解释：细胞群中胞嘧啶分子甲基化的百分比
Although individual methyl groups can either be present or absent on a cytosine, current WGBS data is not from single cells, and we therefore refer to a methylation “level” for each cytosine, interpreted as the fraction of molecules in the underlying cell population that have the methyl mark on the corresponding cytosine.
测序原理判定一个C是否甲基化，所以在reads上一个位点的甲基化水平可以通过甲基化的C数目除以所有C的数量来计算
Since bisulfite sequencing gives a readout for either the presence or absence of a methyl group in each read, we get an unbiased estimate for __the methylation level from the ratio of methylated reads to all reads covering a given site.__ The estimated cytosine methylation level, along with the number of sequenced reads supporting that estimate, is computed for each methylome, along with confidence intervals as described by Hodges et al. [21].

_HMR: hypo methylated regions_
These hypo-methylated regions (HMRs) have been associated with CpG islands, promoter regions, and more generally enhancers and insulators [11]. We developed a highly accurate method for identifying HMRs using a stochastic segmentation model that accounts for both changes in methylation levels and variance in read coverage along the genome [9]. An individual HMR could represent an interval of the genome that has been protected from methylation, or has been opened up by the activity of specific DNA binding proteins [3]. Regardless of the causes, these intervals are strongly associated with regulation of gene expression, and fluctuations in their boundaries between methylomes likely indicate context-specific regulatory sites [10]. In healthy somatic cell methylomes, the organization of methylation features exhibit strikingly precise boundaries. The precision with which HMRs can be identified is what separates this kind of epigenetic information from others, such as H3K4me marks or DNase hypersensitivity sites, that usually lack precision in boundaries.

_Hyper-methylated regions_

In contrast to mammals, the Arabidopsis genome is devoid of methylation by default, with increased methylation levels localized to specific regions, such as intragenic regions and retrotranposons [23]. A similar pattern, referred to as “mosaic methylation” [24], has been documented in invertebrates. During early development of mammalian germ cells, most of the genome is unmethylated, with certain regions retaining methylation despite global epigenetic reprogramming [25]. In each of these cases, the key features of interest are hyper-methylated regions (HyperMR: e.g. Figure 1B), rather than hypo-methylated regions (HMR). Comparative analysis of multiple Arabidopsis methylomes suggests that Arabidopsis HyperMRs, especially those located in intragenic regions, have specific locations that are consistently unmethylated across different ecoytpes and cell types (unpublished studies). These HyperMRs change between different samples in a discrete manner, suggesting that discrete HyperMRs represent a fundamental regulatory and/or functional unit of methylation in plants.

Partially methylated domains.

One important discovery from early genome-wide investigation of the methylomes of immortalized cell lines (e.g. IMR90) was the presence of large partially methylated domains (PMD), that spans hundreds of kilobases [7]. Hansen et al. [12] and Berman et al. [13] also observed such PMDs characteristic of cancer cell lines and primary cancers (e.g. Figure 1C), and PMDs has recently been reported in placenta methylomes as well [26]. PMDs have largely conserved locations across samples, and overlap with nuclear lamina associated domains (LAD) and late replicating regions [13], suggesting their involvement in topological organization of chromosomes. To characterize PMDs we employ a hierarchical method that locates PMDs at low resolution followed by further refinement of their boundaries with higher resolution.


_Allele-specific methylation_

In diploid organisms, the two alleles may have different methylation levels in certain regions (e.g. Figure 1D). Those allele-specific methylated regions (AMRs) can be either parent-of-origin dependent, or associated with allele-specific sequence variation [16]. The former type of allele-specific methylation is related to gene imprinting [27]. We recently developed a novel computational method to identify AMRs using WGBS data without using genotype to separate reads from different alleles [17]. This method allows a comprehensive investigation of allele-specific methylation on a genome-wide scale. By applying this method to public methylomes in MethBase, we have enabled the study of tissue-specific and organism-specific allele-specific methylation.



发现问题：Visual exploration of methylation patterns at specific genes and loci
Visual examination of methylation patterns for specific genes and genomic regions is a valuable research tool, and can be applied to differentially expressed genes, evolutionarily conserved elements, or other regions of interest.

启发性例子：
> Figure 3 shows a genome browser view of the methylation levels and identified HMRs for a subset of human WGBS methylomes in MethBase (disease or mutant samples excluded). The 20 kb genomic interval covers the 5′ end of the DNMT3B gene, a de novo methyltransferase whose absense in mice is embryonic lethal due to developmental defects [30]. Displayed below the partial DNMT3B gene model are transcription factor binding sites (TFBS) identified by the ENCODE project [31]. Among the 26 methylomes displayed below, 9 are from ESCs and iPSCs, and 6 from blood cells, with the rest of diverse cellular origin. In each methylome, an HMR overlaps the transcription start site (TSS) of DNMT3B, 
__but the size of these HMRs and the positioning of their boundaries exhibit interesting variation.__
 In the ESC and iPSC methylomes, as well as 5 of the blood methylomes, this HMR extends downstream of the DNMT3B TSS by roughly 3 kb. In the remaining methylomes the downstream boundary is roughly 1.2 kb downstream of the TSS. The extended portion of this HMR clearly overlaps a cluster of TFBS, suggesting a __regulatory module functioning in iPSC, ESC and blood cells.__ At the other end of this __promoter HMR__ the boundaries seem to form discrete categories, suggesting several __distinct regulatory modules with precise boundaries__. Another HMR appears roughly 10 kb upstream of the DNMT3B TSS exclusively in the iPSC and ESC methylomes. 
__可以协同其他genome-wide数据用于发现特异调控区域结构__
Such these observational features in conjunction with other forms of genome-wide data provide a starting point for dissecting the architecture of particular regulatory regions.












