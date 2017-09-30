---

layout: page
title: NGS基础概念之depth and coverage
categories: definition
tags: [测序常识, NGS基础, 菜鸟团周推]
excerpt_separator: "<!--more-->"
last_modified_at: 2017-09-30T9:30:31-11:30

---

__主要讲三个问题：测序深度和测序覆盖度怎么理解？怎么根据试验要求确定二者数值？一般用什么软件计算？__

<!--more-->
# 理解
## depth 测序深度
__测序深度__是指测序得到的总碱基数与待测基因组大小的比值，可以理解为基因组中每个碱基被测序到的平均次数。测序深度 = reads长度 × 比对的reads数目 / 参考序列长度。假设一个基因大小为2M，测序深度为10X，那么获得的总数据量为20M。

可以根据我们的研究目的来选择相应的测序深度:
![表1](http://mmbiz.qpic.cn/mmbiz/6kvRq2WRXpiaNhZ3TRr6fxoK59BEibL0cq6iaVtwflenGyrQNR0ia9lagqecMIU9FRvGveyfghgibPQsMIxtjAR1Yaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

如果我们的研究目的仅仅是进行群体遗传分析等类型的分析。由于分析是通过计算基因组不同区域的多样性变化来推断进化选择压力，或构建进化树推断遗传分化关系，SNP检出率达到70~90%足以达到此类目的。所以，此类研究常见的测序深度选择在10X左右。

如果想检测个体的全基因组突变，来寻找某个特定功能突变，则我们推荐大于30X的测序深度，以保证接近于“毫无疏漏”。

如果测序样本是混样，例如，癌组织样本（癌细胞的细胞异质性很大），BSA分析中的群体混合池样本。那么在保证30X的测序深度的基础上，如果经费许可50X的测序深度更佳，以保证对混合池中低频变异的检测成功率（例如，新出现或即将消亡的癌细胞突变）。

## coverage 测序覆盖度
__覆盖度__是指测序获得的序列占整个基因组的比例。指的是基因组上至少被检测到1次的区域，占整个基因组的比例。当然，有些文章中也会将测序深度称为Coverage，容易给我们带来混淆。因此，我们还是需要根据语境来推断Coverage的意思。

我们假设基因组大小为G, 假定每次测序可从基因组任何位置上随机检测一个碱基。那么对于基因组上某一个固定碱基位置，在一次测序中，该位置被命中的概率为P （P=1/G）。我们将试验重复n次，相当于产生了n个碱基（n=c*G, c为测序深度）。
碱基的深度分布，相当于求该位置被测到0次，1次，…,n次的概率各是多少? 位点被检测到的n次的概率符合泊松分布。当然，由于概率极低，检测次数很大，所以这样的泊松分布接近于正态分布。

由于基因组中的高GC、重复序列等复杂结构的存在，测序最终拼接组装获得的序列往往无法覆盖有所的区域，这部分没有获得的区域就称为Gap。例如一个细菌基因组测序，覆盖度是98%，那么还有2%的序列区域是没有通过测序获得的。

## 两者关系？
测序深度与基因组覆盖度之间是一个正相关的关系，而测序带来的错误率或变异检测（例如，SNP）假阳性结果会随着测序深度的提升而下降。在图2中可以看到当测序深度达到10x时基因组的覆盖度已接近饱和（左图）。但在测序深度达到10X的时候，SNP的检测率却没有达到饱和。
![](http://mmbiz.qpic.cn/mmbiz/6kvRq2WRXpiaNhZ3TRr6fxoK59BEibL0cq66O6VIDj5j4BSjDtDKGfsHBbaZHXwPU0nUNVJB1EziczYjbophich7Fw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)



# 举例

对长100bp的目标区域进行捕获测序：采用单端测序，每个read长5bp；总共得到了200个reads；把所有的reads比对到目标区域后，100bp的目标区域中有98bp的位置至少有1个read覆盖到，换言之，剩余的2bp没有1个read覆盖。则

深度: 200 x 5 / 100 = 10
我们说这此测序的深度为10X。

覆盖度: 98 / 100 × 100% = 98%
我们说这次测序的覆盖度为98%。

# 应用
在临床panel检测遗传病时，首先要看一下质控，待测的基因是否捕获到？有多少区域大于10X? 很多时候基因检测机构并不提供这么具体的信息，可能会笼统的说XX%覆盖，平均深度是XXX，平均深度对于具体的基因并没有实际意义，另外在科研全外显子测序时，销售可能会说10G数据量，平均100X测序深度，实际结果到底是真是假很难说? 如果我们自己学会分析就可以亲自检验啦。

## 代码

` java -Xmx30g -jar /yourGATKFILE/GenomeAnalysisTK.jar-T DepthOfCoverage -R /reference_genome_dir/ucsc.hg19.fasta -o result_name -Ibam.list -L target.bed --omitDepthOutputAtEachBase --omitIntervalStatistics -ct1 -ct 10 -ct 20 `
Sequencing depth and coverage: key considerations in genomic analyses
GATK 的DepthOfCoverage，-R需要输入你的reference genome，bam.list为待分析的样本，可将多个bam放在一个bam.list里，-L 需要提供捕获的区域，为bed格式，如果是agilent的全外显子可在其官方网站下载，-ct1代表至少覆盖1X的区域，-ct10代表至少覆盖10X的区域，你可以自己添加自己想要的深度。

以上海某知名测序公司的agilent v5外显子捕获测序为例，总数据量为8G左右，结果如下：
![](http://owxb9z5ea.bkt.clouddn.com/17-9-29/26586732.jpg)
 一共4个样本，total代表在捕获区域的总的碱基数，也就是ontarget碱基数，如果我们除以总的测序总量就会得到捕获效率大概为60%左右，也就是40%的数据都是无效的，当然这样的捕获效率在业界算是高的，mean代表平均测序深度，后面是大于1X的捕获区域99.8左右，然后是大于10X的区域为99.3左右，由于人员操作等原因，各公司这一数据的范围在85%-99.5%区间，这一数值对于后面的分析至关重要，如果低于90%就可以说明实验是失败的，再后面是大于20X的区域。
最后再补充一点，NGS所说的10G数据并不是说测序文件在硬盘上的大小为10G，1G代表10亿个碱基，10G代表100亿个碱基。


# 其他资料：
 - 简单视频：https://v.qq.com/x/page/h0367nyz2le.html
        主要介绍如何从suredesign网站下载外显子捕获区域、从ucsc的table browser下载外显子区域，通过GATK的子工具DepthOfCoverage计算覆盖度、大于10X区域比例，大于20X区域比例 

 - 进阶：使用R按照指定宽度计算覆盖率: https://www.plob.org/article/7123.html

 - 2014: Sequencing depth and coverage: key considerations in genomic analyses https://www.nature.com/nrg/journal/v15/n2/pdf/nrg3642.pdf

---
参考资料
[《全基因组重测序深度如何选择?》基迪奥生物](https://mp.weixin.qq.com/s?src=3&timestamp=1506258871&ver=1&signature=jKdw9gNqXgIiG0n7SK9Gyhd5rdMKLkB-xjXkWPzhQICUBh3tZ4lcL7UGXXvZNSj1J53uW20h4Ri6h1xGxfFlZgBos7XNNqQcO2bahY9e7MUuD3cmPs3i0NclolYnEvVmoXijRqPig3XK*x3QpdWafQ==)
https://www.plob.org/article/1217.html
 [《怎样分析panel与全外显子的覆盖度与深度》基因检测与解读](https://mp.weixin.qq.com/s?src=3&timestamp=1506258917&ver=1&signature=X22z2y*tjX88rVER4Xflg19Z4agK5jB70OsTuCZAEJFmdQiAm8SujU3ZgJpCKO106HgQ0yrS3YCGBoY-RWinSpDtl3ISc9w5XPWRxrDQWLV-E2k*zmwTUyfgWohBfQ7LGVzGVzTNe6JbVcSzvvUh3w==)


