---

layout: post
title: '菜鸟团学习Round#2: Bioinformatics Data Skills'
category: codings 
tags: [Reviews, NGS基础, 菜鸟团周推]
excerpt_separator: "<!--more-->"
last_modified_at: 2018-02-27T9:30:31-11:30

---

# Intro: 2018 菜鸟团学习Round2

经过前段时间对基础知识的介绍，小菜鸟已经可以根据一个pipeline 从测序原始数据获得那些量化数据，比如峰值、表达值和甲基化值等等，这些值就被用来做后续分析。但每每当我跑完了原始数据再回看pipeline，发现有很多原来不太明白却因为时间限制而草草掠过的地方还是有很多可以改进的（特别是当老板拿着一些明显的致命错误来问我的时候TnT）。

正所谓“温故而知新”，看似忙碌实则却因各种错误而降低了课题效率，实在是得不偿失于是决定开始在升级这些pipeline的同时系统回顾曾经所学，年初就在菜鸟团里收了这本书：

[Bioinformatics Data Skills](http://shop.oreilly.com/product/0636920030157.do), by [Vince Buffalo](http://www.vincebuffalo.com/)
![](https://covers.oreillystatic.com/images/0636920030157/lrg.jpg)
作者在博客中也指明阅读对象，心内暗喜：

>Bioinformatics Data Skills is an __intermediate-level__ book, aimed at readers with some experience with a scripting language like Python, and very basic Unix (e.g. the Unix filesystem hierarchy, cd, ls, etc.). Bioinformatics Data Skills gives readers a solid Unix foundation in chapters 3 (“Remedial Unix Shell”), 7 (“Unix Data Tools”), and 12 (“Bioinformatics Shell Scripting”, “Writing Pipelines”, and “Parallelizing Tasks”). Readers are also introduced to the R language through learning exploratory data analysis (chapter 8).

<!--more-->


# Main

这本书分成三个部分：

 - __Ideology__: 观念层面上的理解，相当于引言的扩展，主要强调研究的Robust and reproducible，比如实验设计、数据处理时的要注意的细节、态度等等，另外还有一些宝贵的建议，值得多想想自己平时做的和作者说的。这部分理念始终贯穿整本书，值得玩味。
 - __Prerequisite__: 一些必备技能，比如项目的管理方式、服务器进程的管理、远程服务器（现在就是云端了:]），数据的获取和存储，以及最重要的版本控制软件：git！
 -  __Practice__: 
	* unix data tools: know and subtract data (head, grep, cut, awk, uniq), subsell, unix philosophy
	* R : data subsetting, transforming, ggplot, control flow
	* __Range data__: 算是最感兴趣的了，主要就是介绍几个R包，比如GenomicRanges，bedtools等等以及取range时需要注意的
	* Sequencing/Alignment data: 大致就是数据格式、质量和预处理，这在基础介绍的时候有所涉猎了，常见的工具可能会再看看
	* Shell scripting, pipelines, parallelizing tasks: 平时也没怎么修炼的兴趣点又一！
	* 以及最后的存储相关知识: SQL，待要用了再学


数据分析和文献阅读固然重要，高效的基础确实扎实的数据处理基本功！趁此机会，通过这本书好好回顾一下所学，愿在盛夏总结春季所获不至惘然若失！
<p align="right"> 2018初春，王hh </p>

---

# Plans
计划用约九周的时间完成两大块的学习，并chapter1 的思想为核心展开（价值观果然很重要！）～

| week | task |
|------|--------|
| week1 |  Managing a Bioinformatics Project (2) |
| week2 |  Basic shell and remote machines (3&7, 4) |
| week3 |  Git and Data (5~6) |
| week4 |  Review & basic R (1~8) |
| week5 |  Range data (9) |
| week6 |  Sequence data (10) |
| week7 |  Alignment data (11) |
| week8 |  Pipelines and others (12~14) |
| week9 |  Review 9~14 |



