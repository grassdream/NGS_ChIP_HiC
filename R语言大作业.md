# R语言大作业

## 数据下载

[SARS-CoV-2 restructures host chromatin architecture | Nature Microbiology](https://www.nature.com/articles/s41564-023-01344-8#Sec36)

[GEO Accession viewer (nih.gov)](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE179184)

[Run Selector :: NCBI (nih.gov)](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=PRJNA742664&o=sample_name_s%3Aa&s=SRR15000168,SRR15000169,SRR18438215,SRR18438216)

[SRR15000168](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR15000168)mock

[SRR15000169](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR15000169)virus

[SRR18438215](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR18438215)mock

[SRR18438216](https://trace.ncbi.nlm.nih.gov/Traces/sra?run=SRR18438216)virus

在SRA Explorer下载数据

[SRA Explorer (sra-explorer.info)](https://sra-explorer.info/#)

下载数据的命令

```shell
#!/usr/bin/env bash
curl -L ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR150/068/SRR15000168/SRR15000168_1.fastq.gz -o SRR15000168_GSM5411187_A549-ACE2_H3K4me3_Mock_rep1_spiChIP-Seq_Homo_sapiens_ChIP-Seq_1.fastq.gz
curl -L ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR150/068/SRR15000168/SRR15000168_2.fastq.gz -o SRR15000168_GSM5411187_A549-ACE2_H3K4me3_Mock_rep1_spiChIP-Seq_Homo_sapiens_ChIP-Seq_2.fastq.gz
curl -L ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR150/069/SRR15000169/SRR15000169_1.fastq.gz -o SRR15000169_GSM5411188_A549-ACE2_H3K4me3_SARS-CoV-2_rep1_spiChIP-Seq_Homo_sapiens_ChIP-Seq_1.fastq.gz
curl -L ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR150/069/SRR15000169/SRR15000169_2.fastq.gz -o SRR15000169_GSM5411188_A549-ACE2_H3K4me3_SARS-CoV-2_rep1_spiChIP-Seq_Homo_sapiens_ChIP-Seq_2.fastq.gz
nohup curl -L ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR184/015/SRR18438215/SRR18438215_1.fastq.gz -o SRR18438215_GSM5966522_A549-ACE2_Mock2_rep2_Hi-C_Homo_sapiens_Hi-C_1.fastq.gz &
nohup curl -L ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR184/015/SRR18438215/SRR18438215_2.fastq.gz -o SRR18438215_GSM5966522_A549-ACE2_Mock2_rep2_Hi-C_Homo_sapiens_Hi-C_2.fastq.gz &
nohup curl -L ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR184/016/SRR18438216/SRR18438216_1.fastq.gz -o SRR18438216_GSM5966523_A549-ACE2_HI-WA1_rep1_Hi-C_Homo_sapiens_Hi-C_1.fastq.gz &
nohup curl -L ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR184/016/SRR18438216/SRR18438216_2.fastq.gz -o SRR18438216_GSM5966523_A549-ACE2_HI-WA1_rep1_Hi-C_Homo_sapiens_Hi-C_2.fastq.gz &
```

## ChIP-seq

新建了个文件夹，ChIP-seq，在这里分析。

解压所有文件

```shell
解压所有文件
nohup gunzip *.gz &
```

安装FASTQC

[质控软件fastQC的安装及用法 - 简书 (jianshu.com)](https://www.jianshu.com/p/776f1045d21d)

进行质控

```shell
mkdir fastqc_res
nohup fastqc -o ./fastqc_res/ -t 10 SRR15000168_GSM5411187_A549-ACE2_H3K4me3_Mock_rep1_spiChIP-Seq_Homo_sapiens_ChIP-Seq_1.fastq &
nohup fastqc -o ./fastqc_res/ -t 10 SRR15000168_GSM5411187_A549-ACE2_H3K4me3_Mock_rep1_spiChIP-Seq_Homo_sapiens_ChIP-Seq_2.fastq &
nohup fastqc -o ./fastqc_res/ -t 10 SRR15000169_GSM5411188_A549-ACE2_H3K4me3_SARS-CoV-2_rep1_spiChIP-Seq_Homo_sapiens_ChIP-Seq_1.fastq &
nohup fastqc -o ./fastqc_res/ -t 10 SRR15000169_GSM5411188_A549-ACE2_H3K4me3_SARS-CoV-2_rep1_spiChIP-Seq_Homo_sapiens_ChIP-Seq_2.fastq &
```

报错，需要安装java

[(76条消息) Linux系统下安装Java环境（史上最简单没有之一）_linux java 安装_MaoSource的博客-CSDN博客](https://blog.csdn.net/qq_43329216/article/details/118385502)

重新运行没问题，质控很快。

不知道需要做什么过滤处理。

用bwa进行比对

下载bwa

[BWA下载安装 - 简书 (jianshu.com)](https://www.jianshu.com/p/155f03a84e04)

下载参考基因组

```
nohup wget https://hgdownload.soe.ucsc.edu/goldenPath/hg38/bigZips/latest/hg38.fa.gz &
```

建立基因组序列索引

```
nohup bwa index hg38.fa &
```

	[bwa_index] Pack FASTA... 16.40 sec
bwa输出

![image-20230614085824342](https://6465-1316238890.cos.ap-beijing.myqcloud.com/image-20230614085824342.png)

比对

```shell
nohup bwa mem hg38.fa SRR15000168_GSM5411187_A549-ACE2_H3K4me3_Mock_rep1_spiChIP-Seq_Homo_sapiens_ChIP-Seq_1.fastq 1>SRR15000168_ChIP-Seq_1.sam 2>bwa.mem.log1 &

nohup bwa mem hg38.fa SRR15000168_GSM5411187_A549-ACE2_H3K4me3_Mock_rep1_spiChIP-Seq_Homo_sapiens_ChIP-Seq_2.fastq 1>SRR15000168_ChIP-Seq_2.sam 2>bwa.mem.log2 &

nohup bwa mem hg38.fa SRR15000169_GSM5411188_A549-ACE2_H3K4me3_SARS-CoV-2_rep1_spiChIP-Seq_Homo_sapiens_ChIP-Seq_1.fastq 1>SRR15000169_ChIP-Seq_1.sam 2>bwa.mem.log3 &

nohup bwa mem hg38.fa SRR15000169_GSM5411188_A549-ACE2_H3K4me3_SARS-CoV-2_rep1_spiChIP-Seq_Homo_sapiens_ChIP-Seq_2.fastq 1>SRR15000169_ChIP-Seq_2.sam 2>bwa.mem.log4 &
```

安装samtools

[(83条消息) linux下samtools安装_linux装samtools_Gentlezzx的博客-CSDN博客](https://blog.csdn.net/Gentlezzx/article/details/121626879)

自己下载压缩包

[Release 1.17 · samtools/samtools · GitHub](https://github.com/samtools/samtools/releases/tag/1.17)

[(84条消息) Linux 解压tar.bz2格式文件_bz2解压 linux_CAThink的博客-CSDN博客](https://blog.csdn.net/qq_45244708/article/details/105113075)

samtools统计比对结果

```shell
nohup samtools view -S -b SRR15000168_ChIP-Seq_1.sam | samtools flagstat - > flagstat681.txt &

nohup samtools view -S -b SRR15000168_ChIP-Seq_2.sam | samtools flagstat - > flagstat682.txt &

nohup samtools view -S -b SRR15000169_ChIP-Seq_1.sam | samtools flagstat - > flagstat691.txt &

nohup samtools view -S -b SRR15000169_ChIP-Seq_2.sam | samtools flagstat - > flagstat692.txt &
```

差不多都有84%匹对成功。之后详细看看这些文件

比对结果质量统计

```shell
nohup samtools view -b -S -o SRR15000168_ChIP-Seq_1.bam SRR15000168_ChIP-Seq_1.sam &

nohup samtools view -b -S -o SRR15000168_ChIP-Seq_2.bam SRR15000168_ChIP-Seq_2.sam &

nohup samtools view -b -S -o SRR15000169_ChIP-Seq_1.bam SRR15000169_ChIP-Seq_1.sam &

nohup samtools view -b -S -o SRR15000169_ChIP-Seq_2.bam SRR15000169_ChIP-Seq_2.sam &
```



```
nohup samtools sort -o sorted_SRR15000168_ChIP-Seq_1.bam SRR15000168_ChIP-Seq_1.bam &

nohup samtools sort -o sorted_SRR15000168_ChIP-Seq_2.bam SRR15000168_ChIP-Seq_2.bam &

nohup samtools sort -o sorted_SRR15000169_ChIP-Seq_1.bam SRR15000169_ChIP-Seq_1.bam &

nohup samtools sort -o sorted_SRR15000169_ChIP-Seq_2.bam SRR15000169_ChIP-Seq_2.bam &
```

安装qualimap



安装macs2，需要先安装numpy

```shell
pip install MACS2
```

macs2进行peak calling

[MACS2 Call Peak 参数详细学习 - 简书 (jianshu.com)](https://www.jianshu.com/p/53d97099b739)

[如何使用MACS进行peak calling - 简书 (jianshu.com)](https://www.jianshu.com/p/6a975f0ea65a)

```shell
nohup macs2 callpeak -m 10 30 -t SRR15000168_ChIP-Seq_1.sam -f SAM -g hs -n cp_ --nomodel &

nohup macs2 callpeak -m 10 30 -t SRR15000168_ChIP-Seq_2.sam -f SAM -g hs -n cp_ --nomodel &

nohup macs2 callpeak -m 10 30 -t SRR15000169_ChIP-Seq_1.sam -f SAM -g hs -n cp_ --nomodel &

nohup macs2 callpeak -m 10 30 -t SRR15000169_ChIP-Seq_2.sam -f SAM -g hs -n cp_ --nomodel &

报错了

# 常规的peak calling
nohup macs2 callpeak -t sorted_SRR15000168_ChIP-Seq_1.bam -f BAM -g hs -n SRR15000168_1 -B -q 0.01 &

nohup macs2 callpeak -t sorted_SRR15000168_ChIP-Seq_2.bam -f BAM -g hs -n SRR15000168_2 -B -q 0.01 &

nohup macs2 callpeak -t sorted_SRR15000169_ChIP-Seq_1.bam -f BAM -g hs -n SRR15000169_1 -B -q 0.01 &

nohup macs2 callpeak -t sorted_SRR15000169_ChIP-Seq_2.bam -f BAM -g hs -n SRR15000169_2 -B -q 0.01 &

```

sorted_SRR15000168_ChIP-Seq_1.bam
sorted_SRR15000168_ChIP-Seq_2.bam
sorted_SRR15000169_ChIP-Seq_1.bam
sorted_SRR15000169_ChIP-Seq_2.bam

motif分析

使用 bedtools 工具将 MACs 软件得到的 peak calling 结果文件转为 fa 文件。

安装bedtools

[(72条消息) Linux 安装 bedtools2_YKenan的博客-CSDN博客](https://blog.csdn.net/YKenan/article/details/100558781#:~:text=Linux 安装 bedtools2 1 1. 安装依赖 yum install,4 4. 添加环境变量 ... 5 5. 测试 )

```shell
nohup bedtools getfasta -fi hg38.fa -bed SRR15000168_1_peaks.narrowPeak -fo SRR15000168_1_peak.fa -name &

nohup bedtools getfasta -fi hg38.fa -bed SRR15000168_2_peaks.narrowPeak -fo SRR15000168_2_peak.fa -name &

nohup bedtools getfasta -fi hg38.fa -bed SRR15000169_1_peaks.narrowPeak -fo SRR15000169_1_peak.fa -name &

nohup bedtools getfasta -fi hg38.fa -bed SRR15000169_2_peaks.narrowPeak -fo SRR15000169_2_peak.fa -name &
```

使用 MEME 网页工具进行 motif 分析



[学习一遍ChIPseeker的使用 - 简书 (jianshu.com)](https://www.jianshu.com/p/c76e83e6fa57)



## Hi-C

**Appendices**

l **Download HiC data**

curl -L ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR184/015/SRR18438215/SRR18438215_1.fastq.gz -o SRR18438215_GSM5966522_A549-ACE2_Mock2_rep2_Hi-C_Homo_sapiens_Hi-C_1.fastq.gz

curl -L ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR184/015/SRR18438215/SRR18438215_2.fastq.gz -o SRR18438215_GSM5966522_A549-ACE2_Mock2_rep2_Hi-C_Homo_sapiens_Hi-C_2.fastq.gz

curl -L ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR184/016/SRR18438216/SRR18438216_1.fastq.gz -o SRR18438216_GSM5966523_A549-ACE2_HI-WA1_rep1_Hi-C_Homo_sapiens_Hi-C_1.fastq.gz

curl -L [ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR184/016/SRR18438216/SRR18438216_2.fastq.gz -o SRR18438216_GSM5966523_A549-ACE2_HI-WA1_rep1_Hi-C_Homo_sapiens_Hi-C_2.fastq.gz](ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR184/016/SRR18438216/SRR18438216_2.fastq.gz -o SRR18438216_GSM5966523_A549-ACE2_HI-WA1_rep1_Hi-C_Homo_sapiens_Hi-C_2.fastq.gz)

 

l **Unzip fastq. gz data**

gunzip SRR18438215_GSM5966522_A549-ACE2_Mock2_rep2_Hi-C_Homo_sapiens_Hi-C_1.fastq.gz

gunzip SRR18438215_GSM5966522_A549-ACE2_Mock2_rep2_Hi-C_Homo_sapiens_Hi-C_2.fastq.gz

gunzip SRR18438216_GSM5966523_A549-ACE2_HI-WA1_rep1_Hi-C_Homo_sapiens_Hi-C_1.fastq.gz

gunzip SRR18438216_GSM5966523_A549-ACE2_HI-WA1_rep1_Hi-C_Homo_sapiens_Hi-C_2.fastq.gz

 

l **Use bwa for alignment and use samtools to convert sam files to bam files**

bwa mem -SP5M -t8 hg38/hg38.fa fastqc/SRR18438215_GSM5966522_A549-ACE2_Mock2_rep2_Hi-C_Homo_sapiens_Hi-C_1.fastq fastqc/SRR18438215_GSM5966522_A549-ACE2_Mock2_rep2_Hi-C_Homo_sapiens_Hi-C_2.fastq | samtools view -bhs -> SRR18438215.bam

bwa mem -SP5M -t8 hg38/hg38.fa fastqc/SRR18438216_GSM5966523_A549-ACE2_HI-WA1_rep1_Hi-C_Homo_sapiens_Hi-C_1.fastq fastqc/SRR18438216_GSM5966523_A549-ACE2_HI-WA1_rep1_Hi-C_Homo_sapiens_Hi-C_2.fastq | samtools view -bhs -> SRR18438216.bam

 

l **Filtering of bam files with pairtools**

samtools view -h SRR18438215.bam | pairtools parse -c /hg38/hg38.mainonly.chrom.size -o SRR18438215.pairsam

samtools view -h SRR18438216.bam | pairtools parse -c /hg38/hg38.mainonly.chrom.size -o SRR18438216.pairsam

l **Sorting files with pairtools**

pairtools sort --nproc 8 -o sorted.SRR18438215.pairsam SRR18438215.pairsam

pairtools sort --nproc 8 -o sorted.SRR18438216.pairsam SRR18438216.pairsam

 

l **Use pairtools to de-duplicate files**

pairtools dedup --mark-dups -o deduped.SRR18438215.pairsam sorted.SRR18438215.pairsam

pairtools dedup --mark-dups -o deduped.SRR18438216.pairsam sorted.SRR18438216.pairsam

 

l **Filter pairs by pair type**

pairtools select ‘(pair_type==”UU”) or ((pair_type==”UR”) or (pair_type==”RU”)’ -o filtered.SRR18438215.pairsam deduped.SRR18438215.pairsam

pairtools select ‘(pair_type==”UU”) or ((pair_type==”UR”) or (pair_type==”RU”)’ -o filtered.SRR18438216.pairsam deduped.SRR18438216.pairsam

 

l **Split .pairsam file into .pairs file and .sam file**

pairtools split --output-pairs SRR18438215.pairs filtered.SRR18438215.pairsam

pairtools split --output-pairs SRR18438216.pairs filtered.SRR18438216.pairsam

 

l **The obtained .pairs file is compressed to get the gz file**

bgzip SRR18438215.pairs

bgzip SRR18438216.pairs

 

l **Build index**

pairix -f SRR18438215.pairs.gz

pairix -f SRR18438216.pairs.gz

 

l **Use cooler to generate contact matrix**

cooler cload pairix /hg38/hg38.mainonly.chrom.size:50000 SRR18438215.pairs.gz SRR18438215.cool

cooler cload pairix /hg38/hg38.mainonly.chrom.size:50000 SRR18438216.pairs.gz SRR18438216.cool

 

l **Standardization**

cooler balance SRR18438215.cool

cooler balance SRR18438216.cool

###  Hic-Pro

```shell
cd /home/zmj/hic_zyh_out/hic_results/matrix/sample2/raw/500000
cd /home/zmj/hic_zyh_out/hic_results/matrix/sample1/raw/500000/
python /home/zmj/software/HiCPlotter-master/HiCPlotter.py -f sample1_500000.matrix -bed sample1_500000_abs.bed -n test -chr 1 -o test_Chr1 -tri 1 -hmc 5 -mm 10 -ptr 1 -r 500000 -fh 0 -ptd 1

python /home/zmj/software/HiCPlotter-master/HiCPlotter.py -f sample1_500000.matrix -bed sample1_500000_abs.bed -n test -chr 1 -o test_Chr1 -tri 1 -hmc 5 -mm 10 -ptr 1 -r 500000 -fh 0 -ptd 1

python /home/zmj/software/HiCPlotter-master/HiCPlotter.py -f sample1_500000.matrix -bed sample1_500000_abs.bed -n mock_chr9 -chr 9 -o mock_Chr9 -tri 1 -hmc 1 -mm 10 -ptr 1 -r 1000000 -ptd 1 -s 203 -e 264

python /home/zmj/software/HiCPlotter-master/HiCPlotter.py -f sample2_500000.matrix -bed sample2_500000_abs.bed -n covid_chr9 -chr 9 -o covid_Chr9_108 -tri 1 -hmc 1 -mm 10 -ptr 1 -r 1000000 -ptd 1 -s 203 -e 264

python /home/zmj/software/HiCPlotter-master/HiCPlotter.py -f data/HiC/Human/hES-nij.chr21.2 -n hES -chr chr21 -r 40000 -o default2 -ptd 1 -pptd 1 -s 600 -e 900 -fh 0 -w 8 -tr 10 -pi 1

-n 这次任务的名字
-chr 需要被可视化的染色体，如果染色体编号不是Chr*，比如说是Scaffold*，-chr参数需传入对应的值，也可以vi打开将编号修改成Chr
-o 输入文件名，文件路径
-tri 1 默认值为0，1代表着传入的matrix和bed文件是HiC-Pro运行的结果
-r 输入Hi-C结果的分辨率（resolution）
-hmc 热图的颜色 Greys(0), Reds(1), YellowToBlue(2), YellowToRed(3-default), Hot(4), BlueToRed(5)
-ptr 绘制旋转半矩阵（例子中下面的附加图，可以更方便地找到可能的TAD） 
-mm 每一个bin内的互作强度的最大值，默认值为最大值，如果做出来的图片颜色较淡，可以将最大值设的小一点

```

