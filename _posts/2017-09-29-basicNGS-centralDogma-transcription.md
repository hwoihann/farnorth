---

layout: page
title: NGS与转录
category: definition
tags: [中心法则, NGS基础, 菜鸟团周推]
excerpt_separator: "<!--more-->"
last_modified_at: 2017-09-30T9:30:31-11:30

---

[转录（Transcription）](https://en.wikipedia.org/wiki/Transcription_(biology)) 是遗传信息由DNA转换到RNA的过程，即信使RNA（mRNA）以及非编码RNA（tRNA、rRNA等）的合成步骤。在NGS的促进下，研究转录和转录调控的技术有了极大的扩展，以空前的速度发展。
<!--more-->

转录中，一个基因会被读取、复制为mRNA；这个过程由RNA聚合酶（RNA polymerase）和转录因子（transcription factor）所共同完成。两个真核生物转录必备基础名词：
 - [顺式元件](https://en.wikipedia.org/wiki/Cis-regulatory_element)，Cis-regulatory elements (CREs) ：增强或抑制其附近基因转录活性的非编码区域。通常是转录因子结合位点（TFBS）: promoter,  enhancers, silencers, and insulators. 
 - [反式因子](https://en.wikipedia.org/wiki/Trans-regulatory_element)，Trans-regulatory elements：能够特异结合顺式元件的因子，多数为蛋白质，如RNA聚合酶，能和RNA聚合酶结合稳定转录起始复合物的蛋白质等。

![transcription](https://upload.wikimedia.org/wikipedia/commons/thumb/9/9b/MRNA.svg/758px-MRNA.svg.png)


# 1. 转录和转录调控


转录组即某个物种或特定细胞在某一功能状态下产生的所有RNA的总和，可以揭示基因组序列中哪些序列能够表达，而且还能揭示在何时何处表达，以及转录活跃程度。

在每个细胞中，只有一部分基因是能够转录，此处染色质多为开放状态，其余的基因则处于抑制状态，对应的染色质状态较为紧密。在细胞中，保持一类基因的关闭，而另一类基因开启的状态称为基因调控，基因调控是动态的，在生物生长发育中起重要作用。基因表达的调控包含诸多维度比如信号转导，转录前（包括染色质构象和表观调控等），转录调控，转录后调控（剪切、编辑、转运等等），翻译和翻译后调控。其中多个步骤都是围绕着基因序列（DNA序列—传统的认为是顺式元件Cis）和其结合因子包括蛋白和非蛋白因子（传统地认为反式作用因子Trans）而发生。wiki中给出了转录因子可能参与转录调控的所有途径，见下图：
![TF](https://upload.wikimedia.org/wikipedia/commons/thumb/8/80/Transcription_Factors.svg/1024px-Transcription_Factors.svg.png)


基因的表观和转录调控可以分为两个层面，顺式元件和反式因子，技术方法学也可以简单粗暴地分为以研究顺式元件或反式因子为主的两类方法学：

 1. 已知或候选反式因子（表观和转录因子）和顺式元件的研究手段较成熟，比如经典的EMSA，报告基因，ChIP等。 

 2. __如何研究在native状态下与之相互作用的未知反式因子？__ 基因组非编码区域含有大量的组织特异性的调控序列，包括基因转录增强子(enhancer)、沉默子（silencer）、绝缘子（insulators） 等。这些调控序列通常结合几个，几十个，甚至几百个调控因子（包括转录因子，染色质调控蛋白，组蛋白，RNA分子）以及他们形成的三维结构。用来分离纯化单个调控序列的传统的方法面临的最大的挑战是无法区分结合调控序列的特异性调控因子和细胞内大量的非特异性因子。现有的技术包括locked nucleic acids (LNAs)和transcription activator-like (TAL)蛋白只能用在分离纯化多拷贝的基因组重复序列，比如染色体端粒(telomere)。而其他的常规技术，比如ChIP-seq和ChIA-PE则依赖于单个调控因子或组蛋白修饰，而并不能纯化和分析单个调控序列所结合的多个调控因子以及三维结构。在今年8月刚被cell报道的In Situ capture of chromatin interactions by biotinylated dCAS9中，作者首次利用了“biotinylated dCAS9”的方法建立了高分辨率，位点特异原位DNA-蛋白质以及其他元件的互作网络，为攻克这一难题揭开了序幕 [^ref1]。
![](http://owxb9z5ea.bkt.clouddn.com/17-9-29/88002581.jpg)
 [^ref1]: [战胜不可能：转录调控的重大技术进展](https://www.wxwenku.com/d/102384708)



# 2. 转录与NGS


转录组测序的分析流程大致可以分成三类，包括基因组比对（Genome mapping）、转录组比对（Transcriptome mapping）、转录组组装（Reference-free assembly。其中第三种主要是用于分析没有参考基因组和基因注释的物种，应用场合较少且不适合新手入门。对于人、小鼠、大鼠等模式物种，通常用前两种方法进行分析。

转录组测序一般是在你有了一部分生理生化的实验结果，如表型差异、生理指标发生明显变化或有效物质含量出现明显差异等等，在这个基础上你可能会问自己，这些现象内在的机制如何。所以，转录组测序核心回答的是那些基因组存在表达差异，这些存在差异的基因都涉及什么功能，是如何发挥作用的。可以根据实验的目的确定需要转录组测序还是表达谱测序。

## 转录组测序和表达谱测序的区别

 1. 转录组测序 RNA-seq（Transcriptome）
    - 定义： 通过RNA测序，既想得到样本中序列的信息，又需要对序列的表达进行定量和分析。
        - 没有参考基因组的物种，RNA-seq (Transcriptome) 需要进行de novo拼接，对拼接得到的Unigene进行注释；
然后计算de novo得到的Unigene的表达量。所以，这个分析就包含了核酸序列分析和核酸表达定量分析。
        - 对于已有参考基因组的物种，RNA-seq (Transcriptome) 会对测序结果进行基于参考基因组的比对和拼接，从而分析样本中转录本的可变剪切、基因融合、SNP变异等此类针对转录本序列的分析；在序列分析的基础上，在分析转录本的表达量。所以，这个分析也包含了核酸序列的分析和核酸表达量的分析。
    - 对测序的要求：因为涉及到序列的拼接和组装，所以转录组测序对数据量的要求较高（一般单个样本的测序量> 4G），同时一般要求使用双末端测序（Paired end）的数据。[^ref2]
[^ref2]: 转录组测序和表达谱测序的区别》基迪奥生物

 2. 表达谱测序 RNA-seq（Qualification）
    - 定义：Qualification，顾名思义，这个产品的定义就是只对样本中mRNA进行定量分析，而不需要分析mRNA序列的变化。通常此类分析，需要参考序列。参考序列可以是：基因组序列（有参考基因组的物种）或转录组序列（无参考基因组物种，转录组de novo拼接的结果）。
    - 对此类产品的分析要求仅仅是：将测序得到的数据比对到参考序列上，然后计算参考序列的在样本中的对应表达，而不需要去分析参考序列在样本中是否发生了序列变化（可变剪切、基因融合、SNP等）。一般单个样本的测序量2~3G足够，同时单末端测序（single End）和双末端测序（paired End）的数据均可以分析。


# 3. 转录调控与NGS


##常用的转录调控测量技术
参考上文提及的转录因子的调控路径，2012 Shirley Liu 的[Minireview: Applications of Next-Generation Sequencing on Studies of Nuclear Receptor Regulation and Function](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3458226/) 总结得很好，主要有Gene Expression Profiling, Transcription Factor Cistrome Mapping, Epigenome Profiling, Interactions in Three Dimensions, 也直接上图表示：



>![](http://owxb9z5ea.bkt.clouddn.com/17-9-29/71714242.jpg)
Next-generation sequencing applications in studies of transcriptional regulation. Applications in red make use mainly of the quantification of abundance, whereas applications in blue make use of sequence-based observations.

## NGS分析手段

对于可以直接参考HOMER提供的教程，学会了就能分析`ChIP-Seq, GRO-Seq, RNA-Seq, DNase-Seq, Hi-C and numerous other types of functional genomics sequencing data sets`。
入门应不成问题，官网在此<http://homer.ucsd.edu/homer>。

# 总结
转录是在DNA和蛋白质之间传递信息的关键，调控途径多样，分析手段因此也层出不穷，多结合生物学背景，多看文献，才能想到idea再找数据去分析，光会用却不明白相关关系还是不行的。
最后，上一个Homer，住大家国庆快乐，中秋快乐 :metal: ～
![](http://owxb9z5ea.bkt.clouddn.com/17-9-29/5863002.jpg)






