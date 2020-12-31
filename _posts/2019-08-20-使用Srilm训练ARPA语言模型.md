---
title: 使用Srilm训练ARPA语言模型
description: 本文描述了如何使用Srilm工具训练ARPA格式的Ngram语言模型，从编译和安装开始，以几个简单句子为例，描述了模型训练，评价，剪枝，以及插值合并等。
categories:
 - 自然语言
tags:
 - 自然语言
 - 语言模型
---

### 安装SRILM工具

#### 安装tcl

下载tcl安装包：[tcl8.6.8-src.tar.gz](http://www.tcl.tk/software/tcltk/download.html)

执行命令并安装：

```shell
tar -zxvf tcl8.6.8-src.tar.gz
cd tcl8.6.8/
cd unix
make
make install
```

#### 安装srilm

执行命令安装依赖

```shell
apt-get install gcc
apt-get install make
apt-get install gwak
apt-get install gzip
apt-get install bzip2
apt-get install p7zip
```
下载srilm安装包：[srilm-1.7.2.tar.gz](http://www.speech.sri.com/projects/srilm/download.html)  

执行命令并安装：

```shell
pwd
$ /home/sine/
mkdir srilm
mv srilm-1.7.2.tar.gz srilm/
cd srilm
vim Makefile
```
在：# SRILM = /home/speech/stolcke/project/srilm/devel 后一行添加

```sehll
SRILM = $(PWD)
```
执行命令查询机器类型

```shell
uname -i
$ x86_64
```
如果结果为x86_64，则对应的需要修改common/Makefile.machine.i686-m64文件

```shell
vim common/Makefile.machine.i686-m64
```
修改：

```shell
TCL_INCLUDE =
TCL_LIBRARY =
```
为：

```shell
TCL_INCLUDE =
TCL_LIBRARY =

NO_TCL = X
```
修改：

```shell
GAWK = /usr/bin/awk
```
为：

```shell
GAWK = /usr/bin/gawk
````
执行命令并编译srilm

```shell
pwd
$ /home/sine/srilm/
make World
```

执行命令修改环境变量

```shell
pwd
$ /home/sine/
vim .bashrc
```
添加：

```shell
export PATH="/home/sine/srilm/bin/i686-m64:$PATH”
```
并生效环境变量

```shell
pwd
$ /home/sine/
source .bashrc
```
安装完毕！

### 训练模型

训练模型需要文本原始语料，例如speechocean-train.txt，其内容及格式如下：


> 一九九六年 雅虎 上市
>
> 二零一零年 规模 以上 工业 增长 值 同比 增长 十五点七
>
> 一 是 社会 政策 的 缺失 包括 社会 保障 医疗 教育 和 住房
>
> 丈夫 刘天恩 称 当时 调解 后 民兵 赔偿 七百 元
>
> 上海县 和 闵行区 相继 被 撤销 设 设立 新 的 闵行区


#### 词频统计

执行命令获取1gram词频统计

```shell
ngram-count -text speechocean-train.txt -order 1 -write speechocean-train-1gram.count
```
执行命令获取2gram词频统计

```shell
ngram-count -text speechocean-train.txt -order 2 -write speechocean-train-2gram.count
```
执行命令获取3gram词频统计

```shell
ngram-count -text speechocean-train.txt -order 3 -write speechocean-train-3gram.count
```
执行命令获取4gram词频统计

```shell
ngram-count -text speechocean-train.txt -order 4 -write speechocean-train-4gram.count
```

#### Ngram模型训练

执行命令训练1gram语言模型

```shell
ngram-count -read speechocean-train-1gram.count -order 1 -lm speechocean-train-1gram.arpa -interpolate -kndiscount
```

> 其中speechocean-train-1gram.arpa为生成的语言模型，-interpolate和-kndiscount为插值与折回参数

执行命令训练2gram语言模型

```shell
ngram-count -read speechocean-train-2gram.count -order 2 -lm speechocean-train-2gram.arpa -interpolate -kndiscount
```
执行命令训练3gram语言模型

```shell
ngram-count -read speechocean-train-3gram.count -order 3 -lm speechocean-train-3gram.arpa -interpolate -kndiscount
```
执行命令训练4gram语言模型

```shell
ngram-count -read speechocean-train-4gram.count -order 4 -lm speechocean-train-4gram.arpa -interpolate -kndiscount
```

#### 模型剪枝

对3gram语言模型进行剪枝操作

执行命令剪枝3gram模型，剪枝阈值为0.0000001

```shell
ngram -lm speechocean-train-3gram.arpa -order 3 -prune 0.0000001 -write-lm speechocean-train-3gram-pruned-0.0000001.arpa
```
执行命令剪枝3gram模型，剪枝阈值为0.0000003

```shell
ngram -lm speechocean-train-3gram.arpa -order 3 -prune 0.0000003 -write-lm speechocean-train-3gram-pruned-0.0000003.arpa
```

#### 模型质量（困惑度）检查

可以对已经训练的所有模型进行困惑度检查，例如

```shell
ngram -ppl speechocean-train.txt -order 1 -lm speechocean-train-1gram.arpa -debug 2 > speechocean-train-1gram.ppl
```

### 模型文件压缩

执行命令压缩arpa文件，可以节省存储空间。

```shell
gzip speechocean-train-3gram.arpa
$ speechocean-train-3gram.arpa.gz
```

### 模型合并

执行命令，可以将两个已经训练好的arpa模型合并在一起。

```shell
ngram -order 1 -lm speechocean-train-1gram.arpa -mix-lm zhihu-train-1gram.arpa -lambda 0.5 -write-lm combine-train-1gram.arpa
```

> 其中0.5是融合系数。
