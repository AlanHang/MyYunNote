[TOC]

# Hadoop简介

## 1.Hadoop概述

### 1.1 Hadoop是什么

1. Hadoop是一个由Apache基金会所开发的==分布式系统基础框架==。
2. 主要解决海量数据的==存储==和海量数据的==分析计算==问题。
3. 广义上来说，Hadoop通常是指一个更广泛的概念--Hadoop生态圈。

### 1.2 Hadoop三大发行版本

Hadoop三大发行版本：Apache、Cloudera、HortonWorks。

1. Apache版本原始（最基础）的版本。2006
2. Cloudera内部集成了很多大数据框架，对应产品CDH。2008
3. HortonWorks文档较好，对应产品HDP。2011
4. HortonWorks现在已经被Cloudera公司收购，推出新的品牌CDP。2018

1）Apache Hadoop

官网地址：http://hadoop.apache.org

下载地址：https://hadoop.apache.org/releases.html

### 1.3 Hadoop优势

1. 高可靠性：Hadoop底层维护多个数据副本，所以即使Hadoop某个计算元素或存储出现故障，也不会导致数据的丢失。
2. 高扩展性：在集群间分配任务数据，可方便的扩展数以千计的节点。
3. 高效性：在MapReduce的思想下，Hadoop是并行工作的，以加快任务处理速度。
4. 高容错性：能够自动将失败的任务重新分配。

### 1.4 Hadoop组成

![image-20210721150739350](hadoop3.x学习.assets/image-20210721150739350.png)

#### 1.4.1 HDFS概述

HDFS(Hadoop Distribute File System)

NameNode(nn) 存储文件的==元数据==，如文件名，文件目录结果，文件属性（生成时间、副本数、文件权限），以及每个文件的块列表和块所在的DataNode等。

DateNode(dn) 在本地文件系统==存储文件块数据==，以及==块数据的校验和==。

Secondary NameNode(2NN) ==每隔一段时间对NameNode元数据备份==。

#### 1.4.2 Yarn架构概述

1. ResourceManager（RM):管理整个集群资源（内存、CPU等）。

2. NodeManager（NM）：管理单个服务器资源。
3. ApplicationMaster（AM）：单个任务运行的管理。
4. Container：容器，相当于一台独立的服务器，里面封装了任务运行所需要的资源，如内存、CPU、磁盘、网络等。

![image-20210721152258179](hadoop3.x学习.assets/image-20210721152258179.png)

#### 1.4.3 MapReduce架构概述

![image-20210721152553155](hadoop3.x学习.assets/image-20210721152553155.png)

#### 1.4.4 MapReduce、Yarn、MapReduce关系

![image-20210721152926819](hadoop3.x学习.assets/image-20210721152926819.png)

#### 1.4.5 大数据生态体系

![image-20210721153536861](hadoop3.x学习.assets/image-20210721153536861.png)

## 2.Hadoop运行环境搭建

需要安装JDK,配置环境变量(可以在/etc/profile.d/ 路径下生成.sh文件，配置自己的环境变量)

```shell
#java_home
export JAVA_HOME=路径
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
#hadoop_home
export HADOOP_HOME=路径
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin

source /etc/profile
```

![image-20210722104140077](hadoop3.x学习.assets/image-20210722104140077.png)

![image-20210722104321510](hadoop3.x学习.assets/image-20210722104321510.png)

3)配置文件

core-site.xml

![image-20210722104609846](hadoop3.x学习.assets/image-20210722104609846.png)

hdfs-site.xml

![image-20210722104729214](hadoop3.x学习.assets/image-20210722104729214.png)

yarn-site.xml

![image-20210722110035446](hadoop3.x学习.assets/image-20210722110035446.png)

mapred-site.xml

![image-20210722110139924](hadoop3.x学习.assets/image-20210722110139924.png)

4)启动集群

![image-20210722110528279](hadoop3.x学习.assets/image-20210722110528279.png)

![image-20210722110540994](hadoop3.x学习.assets/image-20210722110540994.png)



5）配置历史服务器

![image-20210722112830479](hadoop3.x学习.assets/image-20210722112830479.png)

xsync为自定义分发脚本

6）配置日志聚集

![image-20210722113354542](hadoop3.x学习.assets/image-20210722113354542.png)

7）常用启动命令

![image-20210722113840510](hadoop3.x学习.assets/image-20210722113840510.png)



![image-20210722143751456](hadoop3.x学习.assets/image-20210722143751456.png)

# HDFS

## 1. HDFS概述

### 1.1 HDFS产出背景

![image-20210722144539129](hadoop3.x学习.assets/image-20210722144539129.png)

### 1.2 HDFS优缺点

1. HDFS优点

   ![image-20210722144826800](hadoop3.x学习.assets/image-20210722144826800.png)

2. HDFS缺点

   ![image-20210722145114854](hadoop3.x学习.assets/image-20210722145114854.png)

### 1.3 HDFS组成架构

![image-20210722145508224](hadoop3.x学习.assets/image-20210722145508224.png)

![image-20210722150011578](hadoop3.x学习.assets/image-20210722150011578.png)

### 1.4 HDFS文件块大小

![image-20210722150459442](hadoop3.x学习.assets/image-20210722150459442.png)

![image-20210722152802962](hadoop3.x学习.assets/image-20210722152802962.png)

## 2. HDFS 的Shell操作

![image-20210722152930529](hadoop3.x学习.assets/image-20210722152930529.png)

### 2.1 上传

```shell
#从本地剪切粘贴到HDFS
hadoop fs -moveFromLocal 本地文件 hdfs路径
#从本地文件系统中拷贝文件到HDFS路径
hadoop fs -copyFromLocal 本地文件 hdfs路径
#等同于copyFromLocal
hadoop fs -put 本地文件 hdfs路径
#追加一个文件到已经存在的文件末尾
hadoop fs -appendToFile 本地文件 hdfs文件
```

### 2.2 下载

```shell
#从HDFS拷贝到本地
hadoop fs -copyToLocal hdfs文件 本地路径（可修改文件名）
#等同于copyToLocal
hadoop fs -get hdfs文件 本地路径（可修改文件名）
```

### 2.3 HDFS操作

```shell
#显示目录信息
hadoop fs -ls 文件目录
#显示文件内容
hadoop fs -cat 文件
#修改文件所属权限，和Linux文件系统用法相同
hadoop fs -chmod
hadoop fs -chgrp
hadoop fs -chown
#创建文件路径
hadoop fs -mkdir 文件夹名称
#从HDFS的一个路径拷贝到另一个路径
hadoop fs -cp 源文件 目标路径
#在HDFS目录中移动文件
hadoop fs -mv 源文件 目标路径
#显示一个文件的末尾1kb的数据
hadoop fs -tail 文件
#删除文件或文件夹
hadoop fs -rm 文件
#递归删除目录以及目录文件
hadoop fs -rm -r 文件夹
#统计文件夹的大小信息
hadoop fs -du -s -h 文件夹
#设置HDFS中文件的副本数量
hadoop fs -setrep 副本数量 文件
```

![image-20210722155131888](hadoop3.x学习.assets/image-20210722155131888.png)

![image-20210722155424637](hadoop3.x学习.assets/image-20210722155424637.png)

## 3. HDFS的读写流程

### 3.1 HDFS写入流程

![image-20210722162250068](hadoop3.x学习.assets/image-20210722162250068.png)

**网络拓扑-节点距离计算**

![image-20210722163715380](hadoop3.x学习.assets/image-20210722163715380.png)

机架感知（副本存储节点选择）

**机架感知说明**

![image-20210722164023405](hadoop3.x学习.assets/image-20210722164023405.png)

![image-20210722164119760](hadoop3.x学习.assets/image-20210722164119760.png)

### 3.2 HDFS的读数据流程

考虑节点最近和负载均衡两个方面选择节点。读数据是串行读取（从1到n个数据块串行读取）。

![image-20210722165206387](hadoop3.x学习.assets/image-20210722165206387.png)

## 4.NameNode 和SecondaryNameNode

### 4.1 NameNode和2nn工作机制

![image-20210722170418320](hadoop3.x学习.assets/image-20210722170418320.png)

### 4.3 Fsimage和Edits概念

![image-20210722170827001](hadoop3.x学习.assets/image-20210722170827001.png)

**查看image**

![image-20210722170858368](hadoop3.x学习.assets/image-20210722170858368.png)

**查看编辑日志**

![image-20210722174246559](hadoop3.x学习.assets/image-20210722174246559.png)

## 5. DataNode

**DataNode工作机制**

![image-20210723094328584](hadoop3.x学习.assets/image-20210723094328584.png)

**HDFS配置:hdfs-site.xml**

![image-20210723094347302](hadoop3.x学习.assets/image-20210723094347302.png)

**数据完整性**

![image-20210723094834055](hadoop3.x学习.assets/image-20210723094834055.png)

**DataNode离线时间设置**

![image-20210723095224090](hadoop3.x学习.assets/image-20210723095224090.png)

# MapReduce

## 1. MapReduce概述

定义：

![image-20210723103922361](hadoop3.x学习.assets/image-20210723103922361.png)

优缺点

![image-20210723103857793](hadoop3.x学习.assets/image-20210723103857793.png)

**mapReduce核心思想**

![image-20210723110833959](hadoop3.x学习.assets/image-20210723110833959.png)

**MapReduce进程**

![image-20210723110906547](hadoop3.x学习.assets/image-20210723110906547.png)

**常用的序列化类型**

![image-20210723111406344](hadoop3.x学习.assets/image-20210723111406344.png)

## 2.MapReduce序列化

![image-20210723150529648](hadoop3.x学习.assets/image-20210723150529648.png)

![image-20210723150558214](hadoop3.x学习.assets/image-20210723150558214.png)

## 3. MapReduce框架原理

### 3.1InputFormat数据输入

![image-20210723162724985](hadoop3.x学习.assets/image-20210723162724985.png)

数据切片与MapTask并行度觉得机制

![image-20210723163346481](hadoop3.x学习.assets/image-20210723163346481.png)

**Job提交流程**

![image-20210723165116133](hadoop3.x学习.assets/image-20210723165116133.png)

**FileInputFormat切片源码解析**

![image-20210726142539491](hadoop3.x学习.assets/image-20210726142539491.png)

**FileInputFormat切片大小参数配置**

![image-20210726143033992](hadoop3.x学习.assets/image-20210726143033992.png)

**TextInputFormat方法**

![image-20210726143141193](hadoop3.x学习.assets/image-20210726143141193.png)

**CombineTextInputFormat方法**

![image-20210726143947061](hadoop3.x学习.assets/image-20210726143947061.png)

![image-20210726144228248](hadoop3.x学习.assets/image-20210726144228248.png)

### 3.2 MapReduce工作流程

Reduce端进行数据的拉取。

![image-20210726145822917](hadoop3.x学习.assets/image-20210726145822917.png)

![image-20210726150142932](hadoop3.x学习.assets/image-20210726150142932.png)

### 3.3 Shuffle机制

Map方法之后，Reduce方法之前的数据处理称为Shuffle。

> 排序的方法为快排，对key的索引按照字典进行排序。

![image-20210726150930256](hadoop3.x学习.assets/image-20210726150930256.png)

**Partitioner分区**

![image-20210726153559921](hadoop3.x学习.assets/image-20210726153559921.png)

![image-20210726153749355](hadoop3.x学习.assets/image-20210726153749355.png)

**Combiner合并**

![image-20210726162650454](hadoop3.x学习.assets/image-20210726162650454.png)

![image-20210726162801309](hadoop3.x学习.assets/image-20210726162801309.png)

### 3.4 OutPutFormat

![image-20210726174730974](hadoop3.x学习.assets/image-20210726174730974.png)

### 3.5 源码分析

**MapTask工作机制**

![image-20210727093324372](hadoop3.x学习.assets/image-20210727093324372.png)

**ReduceTask工作机制**

![image-20210727093509160](hadoop3.x学习.assets/image-20210727093509160.png)

**注意事项**

1. ReduceTask = 0 ，表示没有Reduce阶段，输出文件个数和Map个数一致。
2. ReduceTask默认值就是1，所以输出文件为一个。
3. 如果数据分布不均匀，就有可能在Reduce阶段产生数据倾斜。
4. ReduceTask数量并不是任意设置，还要考虑雨雾逻辑需求，有些情况下，需要计算全局汇总结果，就只能有1个ReduceTask。
5. 具体多少ReduceTask，需要根据集群性能而定。
6. 如果分区数不是1，但是ReduceTask为1，是否执行分区过程。答案是：不执行分区过程。因为在MapTask的源码中，执行分区的前提时候先判断ReduceNum个数是否大于1。不大于1肯定不执行。

### 3.7 Reduce Join

![image-20210727105707391](hadoop3.x学习.assets/image-20210727105707391.png)

缺点：

Reduce端处理的方式，合并的操作在Reduce端完成，Reduce端的处理压力太大，Map节点的运算负载则很低，资源利用率不高，且在Reduce阶段极易产生数据倾斜。

### 3.8 Map Join

![image-20210727142723683](hadoop3.x学习.assets/image-20210727142723683.png)

### 3.9 ETL数据清洗

![image-20210727172952595](hadoop3.x学习.assets/image-20210727172952595.png)

## 4. MapReduce 数据压缩

