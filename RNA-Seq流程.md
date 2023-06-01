# RNA-seq流程

# 1.前期准备

在c盘建立文件夹
```
cd /mnt/c
mkdir -p project/rat
cd project/rat
mkdir annotation genome sequence output script

tree
├── annotation
├── genome
├── output
├── script
└── sequence
```
# 2.工具下载
## 2.1 sratoolkit
```
#安装
brew reinstall sratoolkit

#验证是否安装成功
prefetch --help
```
* fastq-dump  

SRA Toolkit中最常用的工具，主要功能是将SRA数据转换为fastq格式。相当于解压缩  

用法：
```
fastq-dump [options] <path/file>
```
* prefetch  

使用方法:
```
 prefetch [options] <SRA accession> [...]
  Download SRA files and their dependencies

  prefetch [options] --cart <kart file>
  Download cart file

  prefetch [options] <URL> --output-file <FILE>
  Download URL to FILE

  prefetch [options] <URL> [...] --output-directory <DIRECTORY>
  Download URL or URL-s to DIRECTORY

  prefetch [options] <SRA file> [...]
  Check SRA file for missed dependencies and download them

```
## 2.2 fastqc

```
#安装
brew reinstall fastqc

#验证是否安装成功
$ fastqc -v
FastQC v0.12.1
```
## 2.3 multiqc
```
# 使用python的安装器安装
pip install 

#验证是否安装成功
multiqc --help
```
## 2.4 cutadapt
```
#安装
pip install cutadapt 

#验证是否安装成功
$ cutadapt --version
4.4
```
## 2.5 质量修剪
* Trim Galore
```
## 需先安装fastqc和cutadapt再安装trim_galore，要安装新版的trim_galore，旧版的报错
cd /mnt/c/project/biosoft
wget https://github.com/FelixKrueger/TrimGalore/archive/0.6.10.tar.gz -o trim_galore.tar.gz
tar zxvf 0.4.5.tar.gz

# 在系统中配置环境变量
vim ~/.bashrc
在界面中输入
# trim_galore
export PATH="/mnt/c/project/biosoft/TrimGalore-0.4.5:$PATH"
#保存后退出并更新
source ~/.bashrc

#验证是否安装成功
trim_galore -v
Quality-/Adapter-/RRBS-/Speciality-Trimming
                                [powered by Cutadapt]
                                  version 0.6.10

                               Last update: 02 02 2023
```
* trimmomatic
```
cd /mnt/c/project/biosoft
wget http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/Trimmomatic-0.38.zip
unzip Trimmomatic-0.38.zip
cd Trimmomatic-0.38

# 导入临时环境变量
export PATH="$(pwd):$PATH"
```



## 2.6 hisat2
```
# 下载安装
wget https://cloud.biohpc.swmed.edu/index.php/s/oTtGWbWjaxsQ2Ho/download

# 在系统中配置环境变量
vim ~/.bashrc
在界面中输入
# hisat2
export PATH="/mnt/c/project/biosoft/hisat2-2.2.1:$PATH"
# 刷新
source ~/.bashrc

# 测试是否可用
$ hisat2 -h
```
## 2.7 samtools
```
#安装
brew install samtools
```
## 2.8 HTseq
```
#安装
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple HTseq
```
## 2.9 R
```
brew install r
```
直接在官网选择``Download R for Windows``; ``install R for the first time``下载安装包即可；注意可以将两个文件放在同一个文件夹内
## 2.10 Rstudio
进入网站：https://www.rstudio.com/products/rstudio/download/
R studio 可以在 Windows 下安装; 选择版本下载,下载完成之后双击安装。
## 2.11 parallel
```
brew install parallel
```
## 3. 数据下载
### 3.1 基因组数据
1. [ensemble网址](https://asia.ensembl.org/)，在左侧五中中选择``Rat``; 在左侧Download DNA sequence (FASTA) 下载基因组序列数据; 在右侧的Download GTF or GFF3 (files for genes, cDNAs, ncRNA, proteins)下载基因注释文件。

可以直接在网页下载，也可用代码
```
# 下载
cd /mnt/d/project/rat/genome
wget http://ftp.ensembl.org/pub/release-107/fasta/rattus_norvegicus/dna/Rattus_norvegicus.mRatBN7.2.dna.toplevel.fa.gz
gzip -d Rattus_norvegicus.mRatBN7.2.dna.toplevel.fa.gz

# 改名（方便后面使用，名字太长一来不方便输入，二来可能会输错）
mv Rattus_norvegicus.mRatBN7.2.dna.toplevel.fa mRatBN7.2.fa

```
下载得到的基因组文件可以查看一下包含哪些染色体，确认文件是否下载正确。
```
cat mRatBN7.2.fa | grep "^>"
```
结果：除了1-20号+X+Y+MT之外还有很多别的ID名。这些都是scaffold
```
>19 dna:primary_assembly primary_assembly:mRatBN7.2:19:1:57337602:1 REF
>20 dna:primary_assembly primary_assembly:mRatBN7.2:20:1:54435887:1 REF
>X dna:primary_assembly primary_assembly:mRatBN7.2:X:1:152453651:1 REF
>Y dna:primary_assembly primary_assembly:mRatBN7.2:Y:1:18315841:1 REF
>MT dna:primary_assembly primary_assembly:mRatBN7.2:MT:1:16313:1 REF
>MU150191.1 dna:primary_assembly primary_assembly:mRatBN7.2:MU150191.1:1:1794995:1 REF

>JACYVU010000493.1 dna:primary_assembly primary_assembly:mRatBN7.2:JACYVU010000493.1:1:444596:1 REF
>MU150193.1 dna:primary_assembly primary_assembly:mRatBN7.2:MU150193.1:1:383091:1 REF
```
每一条primary_assembly的名称后面还跟了一些描述信息，这些描述信息就是当前组装版本，长度等等信息，但是这个信息会妨碍后面写脚本统计或者一些分析，所以这里最好去掉
```
# 首先将之前的名称更改一下
mv mRatBN7.2.fa mRatBN7.2.raw.fa

# 然后去除染色体编号后的描述信息
$ cat mRatBN7.2.raw.fa | perl -n -e 'if(m/^>(.+?)(?:\s|$)/){ print ">$1\n";}else{print}' > mRatBN7.2.fa
#单行匹配，如果匹配到了 开头>多个字母空格或者$，将  >多个字母  打印出来

# 删除
$ rm mRatBN7.2.raw.fa
```
### 3.2 下载注释信息
```
# 下载 gff3 格式
cd /mnt/c/project/rat/annotation
wget http://ftp.ensembl.org/pub/release-107/gff3/rattus_norvegicus/Rattus_norvegicus.mRatBN7.2.107.gff3.gz
gzip -d Rattus_norvegicus.mRatBN7.2.107.gff3.gz
# 同样的也改名
mv Rattus_norvegicus.mRatBN7.2.107.gff3 mRatBN7.2.gff
# 使用head查看部分
head mRatBN7.2.gff
```

```
# 下载 gtf 格式  
cd /mnt/c/project/rat/annotation
wget http://ftp.ensembl.org/pub/release-107/gtf/rattus_norvegicus/Rattus_norvegicus.mRatBN7.2.107.gtf.gz
gzip -d Rattus_norvegicus.mRatBN7.2.107.gtf.gz
```
### 3.3 下载RNA-SEQ数据
```
cd /mnt/c/project/rat/sequence
prefetch SRR2190795 SRR224018{2..7} SRR2240228

#或者这样也可以
cat >1.txt <<EOF
SRR2190795
SRR2240182
SRR2240183
SRR2240184
SRR2240185
SRR2240186
SRR2240187
SRR2240228
EOF
prefetch --option-file 1.txt
```

格式转换

下载得到.sra文件，使用SRAtoolkit工具包的fastq-dump工具，使用它来进行格式转化
```
ls
SRR2190795.sra  SRR2240183.sra  SRR2240185.sra  SRR2240187.sra
SRR2240182.sra  SRR2240184.sra  SRR2240186.sra  SRR2240228.sra

$ parallel -j 4 "    # 用parallel多线程加快速度，并行任务数为4
    fastq-dump --split-3 --gzip {1}    # 将sra文件转化为fastq文件之后压缩为gz文件
" ::: $(ls *.sra)     # :::后接对象

fastq-dump --split-3 --gzip SRR7368844.sra

# ls *.sra代表，列举出任何以.sra结尾的文件
--gzip 将转换出的fastq文件以gz格式输出，可以节省空间
--split-3 把pair-end测序分成两个文件输出，可用于双端测序转化为两个文件，本文举例为单端测序，删掉不影响
-O 输出文件夹名，不加直接放在该文件夹

# 删除sra文件
$ rm *.sra

```
格式介绍
```
# 查看下载好的gz文件
   cd ~/data/sra
   gzip -d -c SRR2190795.fastq.gz | head -n 20

# gzip
-c或--stdout或--to-stdout 　把压缩后的文件输出到标准输出设备，不去更动原始文件。
-d或--decompress或----uncompress 　解开压缩文件。

```
结果：
```
@SRR2190795.1 HWI-ST1147:240:C5NY7ACXX:1:1101:1320:2244 length=100
ATGCTGGGGGCATTAGCATTGGGTACTGAATTATTTTCAGTAAGAGGGAAAGAATCCATCTCCNNNNNNNNNNNNNNNNNNNNNNAAANAAAAATAAAAT
+SRR2190795.1 HWI-ST1147:240:C5NY7ACXX:1:1101:1320:2244 length=100
CCCFFFFFHHHHHJIJJJJJJJJDHHJJJIJJJJJIJJJJJJJJJJJJJJJJJJJJJJJJJHH#####################################
@SRR2190795.2 HWI-ST1147:240:C5NY7ACXX:1:1101:1598:2247 length=100
AACTTCGGTTCTCTACTAGGAGTATGCCTCATAGTACAAATCCTCACAGGCTTATTCCTAGCANNNNNNNNNNNNNNNNNNNNNNTAACAGCATTTTCAT
+SRR2190795.2 HWI-ST1147:240:C5NY7ACXX:1:1101:1598:2247 length=100
@@@7D8+@A:1CFG<C:23<:E<;FF<BHIIEHG:?:??CDF<9DCGGG?1?FEG@@<@CA#######################################
@SRR2190795.3 HWI-ST1147:240:C5NY7ACXX:1:1101:1641:2250 length=100
AGAAGGTCTTAGATCAGAAGGAGCACAGACTGGATGGTCGTGTCATTGACCCTAAAAAGGCTANNNNNNNNNNNNNNNNNNNNNTGAAGAAAATCTTTGT
+SRR2190795.3 HWI-ST1147:240:C5NY7ACXX:1:1101:1641:2250 length=100
BC@FFFDDHHHHHJJJJJJJJJJJJJJJJJJJJIJJJFHGHHEGHIIIHJIJJIJJIJIJJID#####################################
@SRR2190795.4 HWI-ST1147:240:C5NY7ACXX:1:1101:1851:2233 length=100
GGGATTTCATGGCCTCCACGTAATTATTGGCTCAACTTTCCTAATTGTCTGTCTACTACGACANNNNNNNNNNNNNNNNNNNNNNNNNNNNNNTNNCNNN
+SRR2190795.4 HWI-ST1147:240:C5NY7ACXX:1:1101:1851:2233 length=100
@@?DDBDDFFDDDGHGGGGI?B;FFHGHA@FEHGHDDGHEGGFGHIGEHIIHIGGBGACD6AH#####################################
@SRR2190795.5 HWI-ST1147:240:C5NY7ACXX:1:1101:1957:2243 length=100
CAGCCATTGTGGCTCCCGATGGCTTTGACATCATTGACATGACAGCCGGAGGTCAGATAAACTNNNNNNNNNNNNNNNNNNNNNNATCNGTGGCAAAGGT
+SRR2190795.5 HWI-ST1147:240:C5NY7ACXX:1:1101:1957:2243 length=100
@CCFFFFFHHHHAHJJJIJJJJJJIJJIGGGIFIJIIHIIGGJJJJJJJFHIGIJHHHHHHFC#####################################
```
## 4. 质量控制
### 4.1用 fastqc 进行质量评估
1. 具体用法
```
# 基本格式

# fastqc [-o output dir] [--(no)extract] [-f fastq|bam|sam] [-c contaminant file] seqfile1 .. seqfileN

# 主要是包括前面的各种选项和最后面的可以加入N个文件

# -o --outdir FastQC生成的报告文件的储存路径，生成的报告的文件名是根据输入来定的

# --extract 生成的报告默认会打包成1个压缩文件，使用这个参数是让程序不打包

# -t --threads 选择程序运行的线程数，每个线程会占用250MB内存，越多越快咯

# -c --contaminants 污染物选项，输入的是一个文件，格式是Name [Tab] Sequence，里面是可能的污染序列，如果有这个选项，FastQC会在计算时候评估污染的情况，并在统计的时候进行分析，一般用不到

# -a --adapters 也是输入一个文件，文件的格式Name [Tab] Sequence，储存的是测序的adpater序列信息，如果不输入，目前版本的FastQC就按照通用引物来评估序列时候有adapter的残留

# -q --quiet 安静运行模式，一般不选这个选项的时候，程序会实时报告运行的状况。
```
2. 输入代码
```
# 新建目录  
mkdir /mnt/c/project/rat/output/fastqc

！注意！一定在存储fastqc.gz的文件夹路径下执行下面的命令
$ cd /mnt/c/project/rat/output/sequence

fastqc -t 6 -o /mnt/c/project/rat/output/fastqc *.gz
# -t 指定线程数
# -o 指定输出文件夹
# *.gz 表示这个目录下以 .gz 的所有文件
```
3. 结果
```
cd /mnt/c/project/rat/output/fastqc
ls
SRR2190795_fastqc.html  SRR2240183_fastqc.html  SRR2240185_fastqc.html  SRR2240187_fastqc.html
SRR2190795_fastqc.zip   SRR2240183_fastqc.zip   SRR2240185_fastqc.zip   SRR2240187_fastqc.zip
SRR2240182_fastqc.html  SRR2240184_fastqc.html  SRR2240186_fastqc.html  SRR2240228_fastqc.html
SRR2240182_fastqc.zip   SRR2240184_fastqc.zip   SRR2240186_fastqc.zip   SRR2240228_fastqc.zip
```
4. 分析

.html用浏览器打开，查看情况

5. 将所有的fastqc的检测报告合并到一个文件上的程序multiqc，方便分析
```
cd /mnt/c/project/rat/output/fastqc

multiqc .

/// MultiQC 🔍 | v1.12

|           multiqc | Search path : /mnt/d/project/rat/output/fastqc
|         searching | ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 16/16
|            fastqc | Found 8 reports
|           multiqc | Compressing plot data
|           multiqc | Report      : multiqc_report.html
|           multiqc | Data        : multiqc_data
|           multiqc | MultiQC complete

结果: 在该文件夹内得到一个multiqc_report.html文件
```
## 4.2 剔除接头以及测序质量差的碱基

采用trimmomatic进行两端低质量的区域的去除
```
# 新建文件夹
$ mkdir -p ../output/adapter

cd /mnt/c/project/rat/sequence

for i in $(ls *.fastq.gz);
do
cutadapt -a AATGATACGGCGACCACCGAGATCTACACTCTTTCCCTACACGACGCTCTTCCGATCT \
    --minimum-length 30 --overlap 4 --trim-n \
    -o /mnt/c/project/rat/output/adapter/${i}  ${i}
done

# --minimum-length 如果剔除接头后read长度低于30，这条read将会被丢弃
# --overlap        如果两端的序列与接头有4个碱基的匹配将会被剔除
# --trim-n         剔除两端的N
# -a 去除3端引物序列
```
## 4.3 再次去除低质量区域
```
parallel -j 4 "
java -jar /mnt/c/project/biosoft/Trimmomatic-0.38/Trimmomatic-0.38.jar \                                                       SE -phred33 {1} ../trim/{1} \
LEADING:20 TRAILING:20 SLIDINGWINDOW:5:15 MINLEN:30 \
" ::: $( ls *.gz)

# 本命令逻辑  
java -jar  [Trimmomatic软件存储位置]\   #-jar 执行封装在 JAR 存档中的程序
   [单端测序] SE -phred33 {变量名称}      [指明存储路径和名称] ../trim/{变量名称}\
   Trimmomatic软件的命令选项  


# LEADING:20，从序列的开头开始去掉质量值小于 20 的碱基
# TRAILING:20，从序列的末尾开始去掉质量值小于 20 的碱基
# SLIDINGWINDOW:5:15，从 5' 端开始以 5bp 的窗口计算碱基平均质量，如果此平均值低于 15，则从这个位置截断read
# MINLEN:30， 如果 reads 长度小于 30bp 则扔掉整条 read。


#或者运行下面的代码也可以
parallel -j 4 " trimmomatic SE -phred33 {1} ../trim/{1}  LEADING:20 TRAILING:20 SLIDINGWINDOW:5:15 MINLEN:30 " :::$(ls *.gz)
```
结果： # Trimmomatic软件处理转入后台，不会显示实时步骤，耐心等待
## 4.4 再次查看质量情况
```
$ cd /mnt/c/project/rat/output/trim
# 挂载到存储gz文件的文件夹下

$ mkdir ../fastqc_trim
$ parallel -j 4 "
    fastqc -t 4 -o ../fastqc_trim {1}   #不可将1换成i
" ::: $(ls *.gz)

$ cd ../fastqc_trim
$ multiqc .
```
可观察到，比原来的情况好了很多

# 6. 序列比对
## 6.1 建立索引
使用hisat2中的工具hisat2-build建立索引。

* hisat2-build 用法
```
hisat2-build [options]* <reference_in> <ht2_base>
hisat2-build [选项] [基因组序列(.fa)] [索引文件的前缀名]

#<reference_in> ：fasta文件;  如果为list，使用逗号分开
#<ht2_base> ：索引文件的前缀名，如设为xxx，则生成的索引文件为xxx.1.ht2,xxx.2.ht2，默认的前缀名为NAME

```
开始使用 ! 师兄只用一号染色体做了比对，发现结果不太对，所以接下来以全基因组做比对
```
$ cd /mnt/c/project/rat/genome
$ mkdir index
$ cd index
 
hisat2-build  -p 6 ../mRatBN7.2.fa mRatBN7.2
#-p 并行运算线程数为6，-p并行运算线程数(默认:1)。
```
在运行过程中会有部分信息提示，其中说到建立索引文件的分块情况以及运行时间的统计
索引建立完成之后在c:\project\rat\genome\index文件夹下会出现
```
mRatBN7.2.1.ht2
mRatBN7.2.2.ht2
...
mRatBN7.2.8.ht2
```
8个文件，这些文件是对基因组进行压缩之后的文件，这个将基因组序列数据分块成了8份，在执行序列比对的时候直接使用这些文件而不是基因组mRatBN7.2.fa文件。

## 6.2 开始比对
这里使用hisat2进行比对
```
Usage:
  hisat2 [options]* -x <ht2-idx> {-1 <m1> -2 <m2> | -U <r>} [-S <sam>]

  <ht2-idx>  Index filename prefix (minus trailing .X.ht2).
  <m1>       Files with #1 mates, paired with files in <m2>.
             Could be gzip'ed (extension: .gz) or bzip2'ed (extension: .bz2).
  <m2>       Files with #2 mates, paired with files in <m1>.
             Could be gzip'ed (extension: .gz) or bzip2'ed (extension: .bz2).
  <r>        Files with unpaired reads.
             Could be gzip'ed (extension: .gz) or bzip2'ed (extension: .bz2).
  <sam>      File for SAM output (default: stdout)
```

```
# 原来的代码
parallel -k -j 4 "    #-k --keep-order 强制使输出与参数保持顺序；-j 并行任务数
    hisat2 -t -x ../../genome/index/mRatBN7.2 \    # -x 定位到索引文件
      -U {1}.fastq.gz -S ../align/{1}.sam \
      # -U <r>：单端测序list，若为list，使用逗号隔开，-U lane1.fq,lane2.fq,lane3.fq,lane4.fq
      # -S <hit> ：SAM写入的文件名，默认写入到标准输出中

      2>../align/{1}.log      # 标准输出输入到相对应的文件中
" ::: $(ls *.gz | perl -p -e 's/.fastq.gz$//')    # perl 语句起到批量替换作用，将文件格式后缀去掉


# 把索引和测序数据放到同一个文件夹里面
hisat2 -t -x mRatBN7.2 -U SRR2190795.fastq.gz -S SRR2190795.sam

parallel -k -j 2 " 
    hisat2 -t -x mRatBN7.2 \
      -U {1}.fastq.gz -S {1}.sam \
      " ::: $(ls *.gz | perl -p -e 's/.fastq.gz$//')
```


## 正确的做法

* for循环

由于内存占用太大，占满了，程序跑不起来，所以吴师兄帮忙写了一个``for循环``，程序正常运行。我把index索引文件，输入文件和输出文件放到了同一个文件夹里面。（看到网上说在同一个文件夹里面报错的可能性小一些。）
```
guoqinghua@DESKTOP-VPR0E67:/mnt/c/project/rat/output/trim$ for i in $(ls *.gz | perl -p -e 's/.fastq.gz$//')
> do
> hisat2 -t -x mRatBN7.2 -U ${i}.fastq.gz -S ${i}.sam 2>../align/${i}.log
> done
```
* 一个一个单独运行。索引文件和输入输出文件不在同一个文件夹里面。界面输出信息写入到日志中
```
guoqinghua@DESKTOP-VPR0E67:/mnt/c/project/rat/output/trim$ hisat2 -t -x ../../genome/index/mRatBN7.2 -U SRR2190795.fastq.gz -S ../align/SRR2190795.sam 2>../align/SRR2190795.log
```
* 3.一个一个单独运行。
索引文件和输入输出文件不在同一个文件夹里面。界面输出信息未写入到日志中，程序跑完之后在终端界面显示。
```
guoqinghua@DESKTOP-VPR0E67:/mnt/c/project/rat/output/trim$ hisat2 -t -x ../../genome/index/mRatBN7.2 -U SRR2240187.fastq.gz -S ../align/SRR2240187.sam

Time loading forward index: 00:01:51
Time loading reference: 00:00:03
Multiseed full-index search: 00:36:11
19363668 reads; of these:
  19363668 (100.00%) were unpaired; of these:
    1444081 (7.46%) aligned 0 times
    16869438 (87.12%) aligned exactly 1 time
    1050149 (5.42%) aligned >1 times
92.54% overall alignment rate
Time searching: 00:36:15
Overall time: 00:38:06
```
## 6.3 格式转化与排序
原代码
```
parallel -k -j 4 "
    samtools sort -@ 4 {1}.sam > {1}.sort.bam
    samtools index {1}.sort.bam
" ::: $(ls *.sam | perl -p -e 's/\.sam$//')

rm *.sam
```
我输入的代码
```
parallel -k -j 1 "
    samtools sort -@ 1 {1}.sam > {1}.sort.bam
" ::: $(ls *.sam | perl -p -e 's/\.sam$//')

# 有一个没有跑成功，所以单独运行一次
samtools sort SRR2240186.sam > SRR2240186.sort.bam 
```
* samtools sort
```
Usage: samtools sort [options...] [in.bam]
options:
-n : 根据read的name进行排序，默认对最左侧坐标进行排序
-o : 设置排序后输出文件的文件名
-O : 后跟sam或bam，规定排序后输出文件的格式，默认是bam
-@ : 后跟正整数，指定分析所用线程数
-m : 后跟数字 + K/M/G，指定每个线程所使用内存量  
```
* samtools index  
```
 为了能够快速访问bam文件，可以为已经基于坐标排序后bam或者cram的文件创建索引，生成以.bai或者.crai为后缀的索引文件。
 必须使用排序后的文件，否则可能会报错。
 另外，不能对sam文件使用此命令。如果想对sam文件建立索引，那么可以使用tabix命令创建。

index命令格式如下：
      samtools index [-bc] [-m INT] aln.bam |aln.cram [out.index]
      参数：
      -b 创建bai索引文件，未指定输出格式时，此参数为默认参数；
      -c 创建csi索引文件，默认情况下，索引的最小间隔值为2^14，与bai格式一致；
      -m INT 创建csi索引文件，最小间隔值2^INT；
```
输入代码：
```
for i in $(ls *.sort.bam)
   do
   echo ${i}
   samtools index ${i}
   done
```

### 7. 表达量统计
```
cd /mnt/c/project/rat/annotation

# 下载.gtf注释文件
wget http://ftp.ensembl.org/pub/release-107/gff3/rattus_norvegicus/Rattus_norvegicus.mRatBN7.2.107.gff3.gz
```

```
cd /mnt/c/project/rat/output/align
parallel -j 4 "
 htseq-count -s no -f bam {1}.sort.bam ../../annotation/mRatBN7.2.107.gtf \
 >../HTseq/{1}.count 2>../HTseq/{1}.log
 " ::: $(ls *.sort.bam | perl -p -e 's/\.sort\.bam$//')
``` 
```
cd /mnt/c/project/rat/output/HTseq
cat SRR2190795.fastq.gz.count | head -n 10
ENSRNOG00000000001      2
ENSRNOG00000000007      3
ENSRNOG00000000008      0
ENSRNOG00000000009      0
ENSRNOG00000000010      0
ENSRNOG00000000012      0
ENSRNOG00000000017      10
ENSRNOG00000000021      0
ENSRNOG00000000024      844
ENSRNOG00000000033      28
```
### 8. 合并表达矩阵与标准化
#### 8.1 合并

下面使用R语言中的merge将表格合并
```
# 先输入大写R，再输入下列命令；或使用Rstudio，应保证路径一致，目前还不太会
rm(list=ls())
setwd("/mnt/d/project/rat/output/HTseq")

# 得到文件样本编号
files <- list.files(".", "*.count")
f_lists <- list()
for(i in files){
    prefix = gsub("(_\\w+)?\\.count", "", i, perl=TRUE)
    f_lists[[prefix]] = i
}

id_list <- names(f_lists)
data <- list()
count <- 0
for(i in id_list){
  count <- count + 1
  a <- read.table(f_lists[[i]], sep="\t", col.names = c("gene_id",i))
  data[[count]] <- a
}

# 合并文件
data_merge <- data[[1]]
for(i in seq(2, length(id_list))){
    data_merge <- merge(data_merge, data[[i]],by="gene_id")
}

write.csv(data_merge, "merge.csv", quote = FALSE, row.names = FALSE)
```
表格在HTseq文件夹中
#### 8.2 数据标准化

安装R包"GenomicFeatures",在ubuntu中输入
```
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install("GenomicFeatures")
```
选择镜像，任意选择一个中国的都可以，这里我选的 nju.edu.cn
安装R程序包时，出现了一些问题，
然后以管理员身份打开Rstudio，
```
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install("GenomicFeatures")
```
报错提示
```
Warning message:
package(s) not installed when version(s)
  same as or greater than current; use
  `force = TRUE` to re-install:
  'GenomicFeatures'
```
重新输入
```
 BiocManager::install("GenomicFeatures"，force = TRUE)
```
依旧不能正常安装。
需要安装其它的package
```
>library(GenomicFeatures)
Error: package or namespace load failed for ‘GenomicFeatures’ in loadNamespace(j <- i[[1L]], c(lib.loc, .libPaths()), versionCheck = vI[[j]]):
 不存在叫‘prettyunits’这个名字的程辑包

>BiocManager::install("prettyunits")
...
Error in download.file(url, destfile, method, mode = "wb", ...) : 
  cannot open URL 'https://cran.rstudio.com/bin/windows/contrib/4.3/prettyunits_1.1.1.zip'
In addition: Warning message:
In download.file(url, destfile, method, mode = "wb", ...) :
  URL 'https://cran.rstudio.com/bin/windows/contrib/4.3/prettyunits_1.1.1.zip': status was 'SSL connect error'
Warning in download.packages(pkgs, destdir = tmpd, available = available,  :
  下载程序包‘prettyunits’时出了问题
```
可以参考以下网址，修改配置。
Win+R，输入inetcpl.cpl 直接打开Internet选项。打开后，在高级中勾选使用TLS 1.0、使用TLS 1.1、使用TLS 1.2。

但其实修改完，关闭防火墙之后有时也还是不行，多安装几次就可以安装上了，可能是网络不稳定问题。

参考网址：

https://www.cnblogs.com/miyuanbiotech/p/14077003.html

https://www.zhihu.com/question/48323000

R语言中设置工作路径
```
setwd("C:/project/rat/annotation")
```
txdb <- makeTxDbFromGFF("mRatBN7.2.107.gtf",format="gtf")


```
# 先下载 GenomicFeatures
BiocManager::install('GenomicFeatures')
library(GenomicFeatures)

# 构建Granges对象
txdb <- makeTxDbFromGFF("mRatBN7.2.gff3",format="gff3")
# 或者用gtf尝试一下，有什么区别
# txdb <- makeTxDbFromGFF("mRatBN7.2.107.gtf",format="gtf")
#Warning message:
  In .get_cds_IDX(mcols0$type, mcols0$phase) :
  The "phase" metadata column contains non-NA values for features of type
  stop_codon. This information was ignored.

# 查找基因的外显子
exons_gene <- exonsBy(txdb, by = "gene")

# 计算总长度
# reduce()、width()是Irange对象的方法
gene_len <- list()
for(i in names(exons_gene)){
    range_info = reduce(exons_gene[[i]])
    width_info = width(range_info)
    sum_len    = sum(width_info)
    gene_len[[i]] = sum_len
}

# 或者写为lapply的形式(快很多)
gene_len <- lapply(exons_gene,function(x){sum(width(reduce(x)))})

#查看下
class(gene_len)
length(gene_len)
```





```
先把mRatBN7.2_gene_len.tsv"
#将原来储存在annotation中的外显子长度文件粘贴到count 所在的 HTseq 文件夹下 
rm(list=ls())
> setwd("C:/project/rat/output/HTseq")
> gene_len_file <- "mRatBN7.2_gene_len.tsv"
> count_file <- "SRR2190795.count"
> gene_len <- read.table(gene_len_file, header = FALSE, row.name = 1)
> colnames(gene_len) <- c("length")
> count <- read.table(count_file, header = FALSE, row.name = 1)
> colnames(count) <- c("count")
> all_count <- sum(count["count"])
> RPKM <- c()
> for(i in row.names(count)){
+ count_ = 0
+ exon_kb = 1
+ rpkm = 0
+ count_ = count[i, ]
+ exon_kb  = gene_len[i, ] / 1000
+ rpkm    = (10 ^ 6 * count_ ) / (exon_kb * all_count )
+ RPKM = c(RPKM, rpkm)
+ }
```


```
setwd("C:/project/rat/annotation")
library(GenomicFeatures)
txdb <- makeTxDbFromGFF("mRatBN7.2.gff3",format="gff3")
exons_gene <- exonsBy(txdb, by = "gene")

gene_len <- list()
for(i in names(exons_gene)){
range_info = reduce(exons_gene[[i]])
width_info = width(range_info)
sum_len    = sum(width_info)
gene_len[[i]] = sum_len
}
# 或者写为lapply的形式(快很多)
gene_len <- lapply(exons_gene,function(x){sum(width(reduce(x)))})

length(gene_len)
class(gene_len)
data <- t(as.data.frame(gene_len))
write.table(data, file = "mRatBN7.2_gene_len.tsv", row.names = TRUE, sep="\t", quote = FALSE, col.names = FALSE)
```