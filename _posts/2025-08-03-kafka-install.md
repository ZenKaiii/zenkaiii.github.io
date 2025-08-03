---
layout:     post
title:      "Kafka本地安装实践"
subtitle:   "win环境下的kafka安装实操"
date:       2025-08-03 19:00:00
author:     "Zenkai"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Kafka
    - Go
---

> “Yeah It's on. ”

# Kafka本地安装实践

## 1. Kafka安装

下载链接https://archive.apache.org/dist/kafka/.3.5.0/,下载kafka2.12
3.5.0.tgz。
解压到D:\software\kafka2.12-3.5.0(当然也可以解压到其他路径，但这个路径里
不能有空格，路径尽量短一点，否则启动kafka时可能报错：输入命令太长)。
修改config\server.properties,文件：

``````properties
num.partitions=2
log.dirs=D:\\software\\kafka_2.12-3.5.0\\kafka-logs
``````

我们故意把每个Topic对应的partition数目改为2，在go代码里我们分别会演示
Consumer有1个、2个、3个的情况.
这里的log.dirs指的是kafka存储数据的目录，每个topic的partition对应一个子目录，目录名为"{Topic}-{PartitionlD}"。
修改config\zookeeper..properties文件：

``````properties
dataDir=D:\\software\\kafka_2.12-3.5.0\\zookeeper_data
``````

开一个终端，启动zookeeper:（其中zookeeper的启动依赖java环境，因此需要提前安装好jdk，jdk安装参考廖雪峰：https://liaoxuefeng.com/books/java/quick-start/dev-env/install-jdk/index.html）

``````bash
D:\software\kafka 2.12-3.5.0>.\bin\windows\zookeeper-server-start.bat .\config\zookeeper.properties
``````

另开一个终端，启动kafka: （kafka启动依赖WMIC，请确保win系统有安装此功能，若没有请参考：https://blog.csdn.net/weixin_59410575/article/details/144717534）

``````bash
PS D:\softwarelkafka 2.12-3.5.0>.\bin\windows\kafka-server-start.bat
.\config\server.properties
``````



## 2. Kafka实践

本实验使用kafka-go三方组件，应该是目前go语言业界最优解

``````bash
go get github.com/segmentio/kafka-go
``````

编写好kafka producer和consumer，通过go run命令在本地开多终端启动三个consumer和一个producer。

实验发现，当我们启动好三个consumer与一个producer时，发现，其中一个consumer一直未收到消息，另外两个consumer正常接收，这是因为我们刚刚配置的num.partitions=2，代表kafka server只开启两个分区，因此第三个consumer接收不到。

![kafka_consumer](/img/in-post/kafka-demo/kafka_consumer.png)
