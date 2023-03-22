# influxdb1.0 代码

一级目录：

![image-20220919105328692](influxdb.assets/image-20220919105328692.png)

目录解析

- client

  client lib V2版本。

- cmd目录

  influxDb 相关程序所在目录：

  - influxd 目录为influxdb主程序代码；
  - influx 为influxdb自带的控制台管理工具源码；
  - influx_inspect 为influxdb数据查看工具源码；
  - influx_stress 为influxdb压力测试工具源码；
  - influx_tsm 为数据库装换工具（将数据库从b1或者bz1格式转换为tsm1格式）源码；

- coordinator

  协调器，负责数据的写入和一些创建语句的执行。

  在influxdb的changeLog中显示在v1.0.0中使用coordinator转换cluster。

  自建集群功能可以通过此模块实现。

- etc

  存放默认配置

- importer

  数据导出功能，版本向后兼容相关代码。在readme中提到：InfluxDB 0.8.9 版增加了将数据导出为可导入 0.9.3 及更高版本的格式的支持。

  > Version `0.8.9` of InfluxDB adds support to export your data to a format that can be imported into `0.9.3` and later.

- influxql

  influxdb查询语言的解析器。

- internal

  主要实现metaClient接口。

- man

  帮助手册。

- models

  基础数据类型定义。

- monitor

  influxdb系统监控。

- pkg

  一些通用包集合：

  - deep 主要实现deepValueEqual方法，用于深层次比较两个值是否相等；
  - escape 主要实现byte和string两种数据类型转移字符相关操作；
  - limiter 主要是一个基于channel实现的简单并发限制器Fixed；
  - pool 主要实现bytes和generic两种类型的pool，在pool中的对象不使用时不会被垃圾回收自动清理掉；
  - slices 主要实现string数组的操作；

- scripts

  存放关于influxdb的脚本。

- services

  存放关于influxdb的服务。

  - admin 为influxdb内置管理服务；
  - collectd 为 collectd对接服务，可以通过接收UDP发送过来的collectd格式数据；
  - continuous_querier 为influxdb的CQ服务；
  - graphite 为influxdb的graphite服务；
  - httpd 为influxdb的http服务，可以通过该接口进行数据库数据的写入和查询等操作；
  - meta 为influxdb的元数据服务，可以用于管理数据库的元数据相关内容；
  - opentsdb 为influxdb的opentsdb服务，可用于替换opentsdb；
  - precreator 为influxdb的Shard与创建服务；
  - retention 为influxdb的数据保留策略的强制执行服务，主要用于定时删除文件；
  - snapshotter 为influxdb的快照服务；
  - subscribe 为influxdb的订阅服务；
  - udp 为influxdb的udp服务，可以通过改接口进行数据库的写入和查询等操作；

- stress

  压力测试相关内容。

- tcp

  网络连接多路复用

- tests

  测试相关内容。

- toml

  toml解析器，为独立的解析模块，主要解析时间字符串和磁盘容量数据。

- tsdb

  主要是时序数据库的实现。

- uuid

  主要存放uuid生成的相关代码。

# 时序数据库安装

下载安装路径：https://portal.influxdata.com/downloads/ 参考给出的安装步骤即可。

以1.8版本、linux 64系统离线安装为例：

- 下载tar.gz的包 

  `wget https://dl.influxdata.com/influxdb/releases/influxdb-1.8.10_linux_amd64.tar.gz`

- 解压压缩包

  `tar xvfz influxdb-1.8.10_linux_amd64.tar.gz`

- 编辑配置文件

  ```shell
  cd influxdb-1.8.10
  vim influxdb.conf
  
  [meta]
  dir = "/mnt/disk1/influxDB/influxdb/meta"
  [data]
  dir = "/mnt/disk1/influxDB/influxdb/data"
  wal-dir = "/mnt/disk1/influxDB/influxdb/wal"
  seriers-id-set-cache-size = 100
  [coordinator]
  [retention]
  enabled = true
  check-interval = "30m"
  ```

- 启动

  ```shell
  export PATH=$PATH:{安装路径}/usr/bin
  
  #服务端 前台启动
  influxd -config influxdb.conf
  
  #服务端 后台启动
  nohup influxd -config influxdb.conf &2 > 1 > influxdb.out &
  
  #客户端 启动
  influx [-host 主机地址] [-port 端口]
  ```



# 存储引擎



influxdb的存储引擎具有wal和一组只读数据文件，它们在概念上与LSM树中的SSTables类似；TSM文件包含排序，压缩的series数据。

存储引擎将多个组件结合在一起，并提供用于存储和查询series数据的外部接口。 它由许多组件组成，每个组件都起着特定的作用：

- In-Memory Index —— 内存中的索引是分片上的共享索引，可以快速访问measurement，tag和series。 引擎使用该索引，但不是特指存储引擎本身。
  WAL —— WAL是一种写优化的存储格式，允许写入持久化，但不容易查询。 对WAL的写入就是append到固定大小的段中。
- Cache —— Cache是存储在WAL中的数据的内存中的表示。 它在运行时可以被查询，并与TSM文件中存储的数据进行合并。
- TSM Files —— TSM Files中保存着柱状格式的压缩过的series数据。
- FileStore —— FileStore可以访问磁盘上的所有TSM文件。 它可以确保在现有的TSM文件被替换时以及删除不再使用的TSM文件时，创建TSM文件是原子性的。
- Compactor —— Compactor负责将不够优化的Cache和TSM数据转换为读取更为优化的格式。 它通过压缩series，去除已经删除的数据，优化索引并将较小的文件组合成较大的文件来实现。
- Compaction Planner —— Compaction Planner决定哪个TSM文件已准备好进行压缩，并确保多个并发压缩不会彼此干扰。
- Compression —— Compression由各种编码器和解码器对特定数据类型作处理。一些编码器是静态的，总是以相同的方式编码相同的类型; 还有一些可以根据数据的类型切换其压缩策略。
- Writers/Readers —— 每个文件类型（WAL段，TSM文件，tombstones等）都有相应格式的Writers和Readers。

InfluxDB的0.9版本使用BoltDB作为底层存储引擎。下面要介绍的TSM，它在0.9.5中发布，是InfluxDB 0.11+中唯一支持的存储引擎，包括整个1.x系列。

BoltDB，这是一个基于内存映射B+ Tree的引擎，它是针对读取进行了优化的。最后，我们最终建立了我们自己的存储引擎，它在许多方面与LSM树类似。借助我们的新存储引擎，我们可以达到比B+ Tree实现高达45倍的磁盘空间使用量的减少，甚至比使用LevelDB及其变体有更高的写入吞吐量和压缩率。



# InfluxDB文件结构解析

参考文献 https://blog.csdn.net/u012794915/article/details/100061367



# influxDB 简单使用

参考文件 https://jasper-zhang1.gitbooks.io/influxdb/content/Introduction/getting_start.html