MapReduce既是[[Hadoop中]]的模块，也是一个计算模型。用户需要自己将算法划分成Map和Reduce两个阶段。首先将数据划分为小块的数据，将数据分配到不同计算节点的Map任务中计算，然后将计算结果汇总到Reduce节点中进行合并，得出最终结果。MapReduce系统也是主从式的计算系统。

在使用[[YARN]]后，每个集群有一个Resource-Manager，用于管理整个集群。集群中每个计算节点都有一个NodeManager，负责管理某个节点的容器并监视其资源使用。每个应用程序由一个MRAppMaster进行管理。


MapReduce的模型其实非常简单，**其核心思想就是分治。** 

首先，Hadoop会分割输入数据，交给Map任务进行计算，接着将Map的计算结果通过HDFS传递给Reduce任务，由Reduce任务对Map的计算结果进行合并，并将结果输出，得到最后的输出。但这只是一种最简单的描述，其实整个MapReduce的过程中包含了更多的部分，我们可以利用这套机制完成很多类型的计算工作。我们可以用下图来描述完整的MapReduce模型：

![[Pasted image 20211230223134.png]]


首先，Hadoop会从数据源中读取数据。这个数据源类型非常多，可以是文本数据，可以是从HBase或者Cassandra这类数据库中提取的数据，也可以是Avro这种序列化格式。Hadoop提供了一个数据处理类的接口InputFormat，只要用户实现该类，理论上就可以从任何数据源中读取输入，我们常用的有TextInputFormat、KeyValueTextInputFormat、SequenceFileInputFormat等。


接着，Hadoop会对输入进行分割，并将输入分别传送到不同的计算节点上。每个计算节点可以运行多个Map任务，Hadoop会将分割后的结果传送到对应的Map任务中。Hadoop会帮助我们合理安排每台机器上 Map任务的运行数量（但是我们需要设定一个最大值），并且做好计算的负载均衡，以提高整个计算集群的吞吐量。

执行完Map任务之后，Hadoop会将Map的输出结果以用户指定的格式存放在HDFS的特定目录中。在输出之前，Map会将多个任务的执行结果先使用Combine进行一次合并，这样可以减少Reduce的工作负担，很多 时候也可以减少输出的数据量，更充分地提高系统吞吐量。

接下来，Hadoop就会调度Reduce任务对Map的数据进行最后的合并操作。在执行Reduce之前，Hadoop会进行一个Shuffle操作，对Map的输出结果进行特殊处理，比如默认会进行排序。也就是说每一个Reduce任务的输入都是确保有序的。这样我们就可以利用MapReduce进行海量数据的排序工作。


> input -> split -> map -> combine -> shuffle -> reduce -> output(HDFS / database / network)


#### 词频统计案例
下面通过一个单词计数案例来理解各个过程。

首先将文件拆分成splits(片)，并将每个split按行分割形成<key,value>对，如下图所示。这一步由MapReduce框架自动完成，其中偏移量即key值：
![[Pasted image 20211230223916.png]]

将分割好的<key,value>对交给用户定义的map方法进行处理，生成新的<key,value>对，如下图所示：
![[Pasted image 20211230223936.png]]
得到map方法输出的<key,value>对后，Mapper会将它们按照key值进行Shuffle（排序），并执行Combine过程，将key至相同value值累加，得到Mapper的最终输出结果。如下图所示：
![[Pasted image 20211230223959.png]]

Reducer先对从Mapper接收的数据进行排序，再交由用户自定义的reduce方法进行处理，得到新的<key,value>对，并作为WordCount的输出结果，如下图所示:
![[Pasted image 20211230224014.png]]