# ChIP-seq数据分析

## ChIP-seq原理

全称是Chromatin Immunoprecipitation，也就是染色质免疫共沉淀。

作用是识别蛋白质与DNA的相互作用情况。

### 测序流程

一句话概述：首先通过交联把DNA和蛋白质固定住，接下来降解，再用特异性抗体与特定的蛋白质结合，保留抗体结合的DNA和蛋白质的复合物，去交联保留DNA，接下来扩增测序。

DNA和蛋白质交联(cross-linking)，超声(sonication)将染色体随机切割，利用抗原抗体的特异性识别(IP)，把**目标蛋白**相结合的DNA片段沉淀下来，反交联释放DNA片段，最后是测序(sequencing)。**—Y叔**

![图片](https://6465-1316238890.cos.ap-beijing.myqcloud.com/640)

下图是DNA和转录因子（或者其他蛋白质）的结合状态：

![image-20230619133314030](https://6465-1316238890.cos.ap-beijing.myqcloud.com/image-20230619133314030.png)

接下来，使用交联剂（如甲醛）把转录因子和DNA进行交联，交联的意思是让他们粘在一起，否则之后分离了就检测不到了。、

![image-20230619133439107](https://6465-1316238890.cos.ap-beijing.myqcloud.com/image-20230619133439107.png)

接下来用声波讲解发将其断为碎片。

![image-20230619133514501](https://6465-1316238890.cos.ap-beijing.myqcloud.com/image-20230619133514501.png)

再对某种转录因子用特异性抗体进行结合，比如下图抗体都和红色的转录因子结合。不过也有出错的地方，比如那个蓝色的。

![image-20230619133637617](https://6465-1316238890.cos.ap-beijing.myqcloud.com/image-20230619133637617.png)

免疫共沉淀，保存和抗体结合的DNA和蛋白质复合物。

![image-20230619133733136](https://6465-1316238890.cos.ap-beijing.myqcloud.com/image-20230619133733136.png)

去交联和DNA纯化，解除交联，只保留DNA。

![image-20230619133804125](https://6465-1316238890.cos.ap-beijing.myqcloud.com/image-20230619133804125.png)

扩增DNA然后开始测序。

![image-20230619133918580](https://6465-1316238890.cos.ap-beijing.myqcloud.com/image-20230619133918580.png)

## 分析流程

### 概述

首先质量控制，看看测序的质量如何，把数据不好的给过滤掉。接着做比对，获得这些DNA片段再染色体上的位置信息。接着就是找峰（peak calling），接下来就得到了峰（peak），这个峰也就是蛋白结合位点。接下来就是下游分析，比如可视化、发现motif、注释和功能富集等。

![img](https://6465-1316238890.cos.ap-beijing.myqcloud.com/8242255-ed9f825cf93a7fa8.png)

### 质控过滤比对

这些比较常规

```shell
nohup fastqc -o ./fastqc_res/ -t 10 SRR15000168_GSM5411187_A549-ACE2_H3K4me3_Mock_rep1_spiChIP-Seq_Homo_sapiens_ChIP-Seq_1.fastq &
nohup wget https://hgdownload.soe.ucsc.edu/goldenPath/hg38/bigZips/latest/hg38.fa.gz &
nohup bwa index hg38.fa &
nohup bwa mem hg38.fa SRR15000168_GSM5411187_A549-ACE2_H3K4me3_Mock_rep1_spiChIP-Seq_Homo_sapiens_ChIP-Seq_1.fastq 1>SRR15000168_ChIP-Seq_1.sam 2>bwa.mem.log1 &
nohup samtools view -S -b SRR15000168_ChIP-Seq_1.sam | samtools flagstat - > flagstat681.txt &
```

### peak calling

使用MACS2进行找峰。下面我对命令进行说明。徐州更大佬写的博客非常棒！

#### 命令详解

[如何使用MACS进行peak calling - 简书 (jianshu.com)](https://www.jianshu.com/p/6a975f0ea65a)

```shell
nohup macs2 callpeak -t sorted_SRR15000168_ChIP-Seq_1.bam -f BAM -g hs -n SRR15000168_1 -B -q 0.01 &
```

- `-t/--treatment FIELNAME`和`-c/--control FILENAME`表示处理样本和对照样本输入。其中`-t`必须。这里我没有对照组，所以只有一个上面得到的bam文件。
- `-f/--format FORMAT`用来声明输入的文件格式，目前MACS能够识别的格式有 "ELAND", "BED", "ELANDMULTI", "ELANDEXPORT", "ELANDMULTIPET" (双端测序), "SAM", "BAM", "BOWTIE", "BAMPE", "BEDPE". 除"BAMPE", "BEDPE"需要特别声明外，其他格式都可以用`AUTO`自动检测。我这里是BAM文件。
- `-g`表示实际可比对的基因组大小。比如说人类是2.7e9，也就是2.7G，而实际人类基因组大概是3.2G左右。这是因为有些地方无法拼接，会用N代替，这部分区域大概是80%左右。
- `-n/--name`表示实验的名字, 请取一个有意义的名字。这里我就取了id号。
- `-B/--bdg`: 以bedGraph格式存放*fragment pileup*, *control lambda*, *-log10pvalue* 和*log10qvale*.
  - `NAME_treat_pileup.bdg`: 处理后数据
  - `NAME_control_lambda.bdg`： 对照的局部lambda值
  - `NAME_treat_pvalue.bdg`： 泊松检验的P值
  - `NAME_treat_qvalue.bdg`：Benjamini–Hochberg–Yekutieli处理后的Q值
- `-q`： q值(最小的FDR)的阈值，默认0.05。可以根据结果进行修正。q值是p值经Benjamini–Hochberg–Yekutieli修正后的值。

#### 输出文件

- NAME_peaks.xls: 以表格形式存放peak信息，虽然后缀是xls，但其实能用文本编辑器打开，和bed格式类似，但是以1为基，而bed文件是以0为基.也就是说xls的坐标都要减一才是bed文件的坐标
- NAME_peaks.narrowPeak NAME_peaks.broadPeak 类似。后面4列表示为， integer score for display， fold-change，-log10pvalue，-log10qvalue，relative summit position to peak start。内容和NAME_peaks.xls基本一致，适合用于导入R进行分析。
- NAME_summits.bed：记录每个peak的peak summits，换句话说就是记录极值点的位置。MACS建议用该文件寻找结合位点的motif。能够直接载入UCSC browser，用其他软件分析时需要去掉第一行。
- NAME_peaks.gappedPeak: 格式为BED12+3，里面存放broad region和narrow peaks。
- NAME_model.r，能通过`$ Rscript NAME_model.r`作图，得到是基于你提供数据的peak模型。
- .bdg文件能够用UCSC genome browser转换成更小的bigWig文件。

> 作者：xuzhougeng
> 链接：https://www.jianshu.com/p/6a975f0ea65a
> 来源：简书