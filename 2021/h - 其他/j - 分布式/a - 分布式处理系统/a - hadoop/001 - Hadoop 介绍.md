# 001 - Hadoop 介绍

Hadoop是由Apache基金会开发的分布式存储与计算框架。用户不需要了解底层的分布式计算原理就可以轻松开发出分布式计算程序，可以充分利用集群中闲置的计算资源，将集群的真正威力调动起来。

Hadoop由两个重要模块组成。一个是Hadoop分布式文件系统（Hadoop Distributed File System HDFS），顾名思义，就是一个分布式的文件系统，可以将文件数据分布式地存储在集群中的不同节点上。另一个是MapReduce系统，是一个针对大量数据的分布式计算系统。而当前版本的Hadoop中加入了一个名为[[YARN]]的模块，这是一个用于任务调度和资源管理的框架，而目前MapReduce便是基于YARN开发的。

总结：
Hadoop = [[HDFS]] + [[MapReduce]]


Hadoop已经成为目前世界上最流行的分布式计算框架，Apache也建立了不少与Hadoop相关的项目，如HBase、Cassandra、Avro、Hive、Mahout等项目。


### Apache Hadoop特性

Apache Hadoop具有以下几个特点：

-   高可靠性：Apache Hadoop可以可靠地将数据存储到节点上。
    
-   高可扩展性：Apache Hadoop的存储和计算节点可以快速扩展，并自动进行负载均衡。
    
-   高效性：一方面Apache Hadoop会自动在各个节点之间动态调动数据，保证每个节点存储均衡，另一方面读取数据时我们可以从不同节点并行读取，提高数据读取的速度。
    
-   高容错性：Apache Hadoop会将数据冗余存储在不同节点上，保证数据容错性，计算任务失败时也会自动重新分配任务。
    
-   低成本：一方面，Apache Hadoop是开源软件，可以节省商业软件的购买成本。同时，Apache Hadoop可以用廉价节点组成的集群取代昂贵的超级计算机，从而可以节省硬件成本。


  
Apache Hadoop虽然是异常可靠的分布式计算框架，但其计算存储模型也导致它的严重缺陷——**实时性较差**。首先MapReduce计算模型本身是一种**批处理**的计算模型，也就是积累一批数据，然后启动MapReduce任务处理完这一批数据，等到下次积累到一定程度，再定时或手动启动一轮新任务，而不是随着数据到来即时处理。

此外，HDFS不是一个高实时性的分布式文件系统。为了提高其实时性我们还需要自己加上很多**缓存优化。** 而致命问题在于MapReduce各个任务之间的通信完全使用HDFS完成，这也就从根本上导致MapReduce不可能具有极高的实时性。



### 四、安装 Apache Hadoop

参考 [林子雨编著《大数据技术原理与应用》教材配套大数据软件安装和编程实践指南](http://dblab.xmu.edu.cn/post/5663/)

#### 1、Hadoop 安装

参考 [大数据处理架构Hadoop 学习指南](http://dblab.xmu.edu.cn/blog/285/)

参考 [Hadoop安装教程_单机/伪分布式配置_Hadoop2.6.0/Ubuntu14.04](http://dblab.xmu.edu.cn/blog/install-hadoop/)

参考 [Hadoop安装教程_伪分布式配置_CentOS6.4/Hadoop2.6.0](http://dblab.xmu.edu.cn/blog/install-hadoop-in-centos/)

#### 2、HDFS 安装

参考 [分布式文件系统HDFS 学习指南](http://dblab.xmu.edu.cn/blog/290-2/)

参考 [查看HDFS文件系统数据的三种方法](http://dblab.xmu.edu.cn/blog/893-2/)

#### 3、HBase 安装

参考 [分布式数据库HBase 学习指南](http://dblab.xmu.edu.cn/blog/install-hbase/)

### 五、MapReduce 使用介绍

参考 [MapReduce 学习指南](http://dblab.xmu.edu.cn/blog/631-2/)

参考 [使用Eclipse编译运行MapReduce程序_Hadoop2.6.0_Ubuntu/CentOS](http://dblab.xmu.edu.cn/blog/hadoop-build-project-using-eclipse/)

参考 [使用命令行编译打包运行自己的MapReduce程序 Hadoop2.6.0](http://dblab.xmu.edu.cn/blog/hadoop-build-project-by-shell/)