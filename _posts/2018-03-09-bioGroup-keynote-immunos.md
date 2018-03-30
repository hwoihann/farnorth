---

layout: post
title: '癌症相关数据库: Keynote Ontology Clinical Trials'
category: definition 
tags: [癌症相关数据库专题, 菜鸟团周推]
excerpt_separator: "<!--more-->"
last_modified_at: 2018-03-09T21:30:31-22:10

---

# Pembrolizumab

## 1.1 介绍

Pembrolizumab是人源化单克隆抗体，首个被批准的阻断PD-1的靶向治疗药物。
PD-1全名是程序性死亡分子1，英文名叫programmeddeath-1(PD-1)，是近年来发现的一种负性共刺激分子。PD-L1是PD-1的配体，各种肿瘤细胞组织中都有PD-L1的表达。 PD-1与PD-L1结合后可提供抑制性信号，诱导T细胞凋亡，抑制T细胞的活化和增殖，进而降低了免疫系统对抗肿瘤细胞的能力。

<!--more-->
pembrolizumab作用机制

![](http://owxb9z5ea.bkt.clouddn.com/18-3-5/74854260.jpg)

## 1.2 简史

Pembrolizumab 已于2014年被美国FDA授予“突破性治疗药物”资格，批准用于局部晚期的或转移性黑色素瘤，以及用于晚期非小细胞肺癌。在已经进行的临床研究 中，Pembrolizumab的表现十分令人欣喜。655例晚期黑色素瘤患者用了Pembrolizumab后，结果发现，单药达到33%的有效率，无 进展生存率35%，中位总生存期23个月。Pembrolizumab 作为一线治疗药物用于晚期黑色素瘤优于标准治疗药物伊匹单抗，在两款免疫检查点抑制剂一对一的对照试验 中，Pembrolizumab(Keytruda) 达到了其所有的主要生存期终点，患者缓解率为 33%，几乎是伊匹单抗 12% 缓解率的三倍。
英国国家卫生保健优化研究所推荐将 Pembrolizumab 作为一种治疗药物用于一些晚期、不可切除或转移性黑色素瘤患者。单一治疗药物已在英国获得上市许可，用于晚期（不可切除或转移性）黑色素瘤成年患者治疗。
![](http://owxb9z5ea.bkt.clouddn.com/18-3-5/2017934.jpg)

## 1.3 _扩展：PD-1抑制剂_

PD-1抑制剂，包括PD-1抗体和PD-L1抗体，是一类免疫治疗的新药。主要的作用机制，是阻断PD-1和PD-L1之间的相互作用，因为这两个蛋白的相互作用，会帮助肿瘤逃脱免疫系统的追杀。PD-1/PD-L1抗体，通过阻断这种罪恶的连接，促进病人自身的免疫系统杀伤肿瘤。

PD-1抗体有两种：Nivolumab（O药）适应症：黑色素瘤、非小细胞肺癌、肾细胞癌、经典型霍奇金淋巴瘤、头颈鳞癌、膀胱癌、胃癌、肝细胞癌、携带微卫星不稳定或错配修复基因缺陷的实体瘤。Pembrolizumab（K药）适应症：黑色素瘤、非小细胞肺癌、头颈鳞癌、经典型霍奇金淋巴瘤、膀胱癌、肝细胞癌、胃癌、携带微卫星不稳定或错配修复基因缺陷的实体瘤。PD-L1抗体三种：Atezolizumab（T药）适应症：膀胱癌、非小细胞肺癌。Avelumab（B药）适应症：膀胱癌、Merkel细胞癌。Durvalumab（I药）适应症：膀胱癌。

言归正传， Keynote Ontology Clinical Trials与Pembrolizumab有什么关系呢？
---

# Keynote Ontology Clinical Trials

## 2.1 平台的作用和由来

Keynote Ontology Clinical Trials(https://keynoteclinicaltrials.com/)是用Keytruda进行癌症免疫疗法的临床试验的介绍平台，介绍了若干适用PD-1疗法的癌症临床试验信息，主要供患者参考和报名相关的临床试验。所以目测“keynote”的名字应该是由“keytruda”而来。


## 2.2 可参与临床试验的癌症分类
- Bladder
- Breast
- Colorectal
- Esophageal
- Gastric
- Head and Neck
- Hematology
- Kidney
- Liver
- Lung
- Melanoma
- Ovarian
- Pediatric
- Prostate
- Solid Tumors

每个癌症目录下都有一些典型癌症的描述，比如三阴型乳腺癌，拷贝数异常的结肠癌等等，以及一段“实验参与者告知书”和当前活动的临床试验（下表）
![](http://owxb9z5ea.bkt.clouddn.com/18-3-5/17390319.jpg)

其中每个title的含义：
 - 1: keynote 临床试验的title，比如keynote355，点进去就是对该项目的具体描述。
![](http://owxb9z5ea.bkt.clouddn.com/18-3-5/19523115.jpg)
 - 2: 记录的参与人数
 - 3: 癌症的类型
 - 4: phase，药物临床试验的几个阶段
![](http://owxb9z5ea.bkt.clouddn.com/18-3-5/42635917.jpg)
 - 5: 招募状态，招募中的话就进入keynote的页面，而完成招募项目进行中的跳转页面则是[geo clinical](https://clinicaltrials.gov/ct2/home ) 的页面。比如keynote119，有一个专属号码：MK-3475-119/KEYNOTE-119 (https://clinicaltrials.gov/ct2/show/NCT02555657)，MK-3475就是keytruda了，记录了试验的起止日期、生存时间（多数是保密数据）等信息。

>Participating in a clinical trial is an important decision. Anyone participating in a trial should know as much as possible about what is being studied, what risks are involved and what potential benefits may be gained before deciding to enroll. If you are interested in participating in a clinical trial, please talk to your doctor first.


## 2.3 Clinical Trial的FAQ

记录了一些参与试验的注意事项，常见的问题等等。
https://keynoteclinicaltrials.com/clinical-trials-faq#accordion-4
![](http://owxb9z5ea.bkt.clouddn.com/18-3-5/40464681.jpg)

一个药物就有这么系统完整的医疗记录，很值得国内借鉴啊！

---

REF
http://databankcollect6.blogspot.kr/2016/01/pembrolizumabmk-3475.html

