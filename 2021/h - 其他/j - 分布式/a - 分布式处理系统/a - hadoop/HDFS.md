Hadoop分布式文件系统（HDFS）是一个主从式的分布式文件系统，是GFS的一种开源实现。

HDFS可以利用大量廉价存储器组成分布式存储集群，取代昂贵的集中式磁盘存储阵列。而HDFS集群由一个[[NameNode]]和多个[[DataNode]]组成，除此之外还有用于热备份的[[Secondary NameNode]]，防止集群出现单点故障。