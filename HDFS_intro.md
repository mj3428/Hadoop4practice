# HDFS
HDFS是Hadoop的旗舰级文件系统
## HDFS的设计
以流式数据访问模式来存储超大文件  
* 流式数据访问 HDFS思路：一次写入、多次读取是最高效的访问模式
## HDFS的概念
### 数据块
磁盘有块概念，是磁盘进行数据读写的最小单位，文件系统块一般为几千字节，而磁盘块一般为512字节。  
HDFS也有block概念，大得多，默认为128MB，HDFS上的文件也被划分为块大小的多个分块(chunk)，作为独立的存储单元。但与磁盘不同，
HDFS中小于一个block的文件不会占据整个块的空间。  
**目的：** 最小化寻址开销  
与磁盘文件系统相似，HDFS中的fsck指令可以显示块信息  
`hdfs fsck / -files -blocks`  
### namenode和datanode
管理节点(namenode)-工作节点(datanode)工作模式；namenode管理文件系统的命名空间。它维护者文件系统树及整棵树内所有的文件和目录。
这些信息以两个文件形式永久保存在本地磁盘上：命名空间镜像文件和编辑日志文件。namenode也记录着每个文件中各个块所在的数据节点信息，
但它并不永久保存块的位置信息，因为这些信息会在系统启动时根据数据节点信息重建。  
**namenode非常重要！！** 因此要有容错机制：  
* 1.备份那些组成文件系统元数据持久状态的文件。Hadoop可以通过配置使namenode在多个文件系统上保存元数据的持久状态。这些写操作是
实时同步的，且是原子操作。一般的配置是，在持久状态写入本地磁盘的同时，写入一个远程挂载的网络文件系统NFS。  
* 2.运行一个辅助namenode，但它不能被用作namenode。这个辅助namenode的重要作用是定期合并编辑日志与命名空间镜像，以防止编辑日志
过大。这个辅助namenode一般在另一台单独的物理计算机上运行，因为它需要占用大量CPU时间，并且需要与namenode一样多的内存来执行合并
操作。它会保存合并后的命名空间镜像的副本，并且在namenode发生故障时启用。但是，辅助namenode保存的状态总是滞后于主节点，所以
在主节点全部失效时，难免会丢失部分数据。  
### 块缓存
通常datanode从磁盘中读取块，但对于访问频繁的文件，其对应的块可能被显式地缓存在datanode的内存中，以堆外块缓存(off-heap block cache)
的形式存在。默认情况，一个块仅缓存在一个datanode的内存中，当然可以针每个文件配置datanode的数量。作业调度器（用于MapReduce、Spark
和其他框架的）通过在缓存块的datanode上运行任务，可以利用块缓存的优势提高读操作的性能。缓存池是一个用于管理缓存权限和资源使用的管理
性分组。  
### 联邦HDFS
联邦HDFS允许通过添加namenode实现扩展命名空间中的一部分。例如，一个namenode可能管理/user目录下的所有文件，而另一个namenode可能管理
/share目录下的所有文件。  
联邦环境下，每个namenode维护一个命名空间卷(namespace volume),由命名空间的元数据和一个数据块池(block pool)组成，数据块池包含该命名
空间下文件的所有数据块。命名空间卷之间是相互独立的，两两之间并不相互通信，甚至其中一个namenode的失效也不会影响由其他namenode维护的命名
空间的可用性。
### HDFS高可用性
新的namenode知道满足一下情形才能响应服务:  
1. 将命名空间的映像导入内存中
2. 重演编辑日志
3. 接收到足够多的来自datenode的数据块报告并退出安全模式  

对于一个大型并拥有大量文件和数据块的集群，namenode的冷启动需要30分钟，甚至更长  
两种高可用性共享存储:**NFS过滤器或群体日志管理器QJM**  
QJM是一个专用的HDFS实现，为提供一个高可用的编辑日志而设计，QJM以一组日志节点的形式运行，每一次编辑必须写入多数日志节点。  
### 故障切换与规避
同一时间QJM仅允许一个namenode向编辑日志中写入数据。有时，设置一个SSH规避命令用于杀死namenode的进程是一个好主意。当使用
NFS过滤器实现共享编辑日志时，由于不可能同一时间只允许一个namenode写入数据（这也是为什么推荐QJM的原因），需要更有力的规避方法。
规避机制包括:撤销namenode访问共享存储目录权限、通过远程管理命令屏蔽相应的网络端口。  
## 命令行接口
伪分布配置时，有两个属性项需要进一步解释。  
第一项： s.defaultFS,设置为hdfs://localhost/,用于设置Hadoop默认文件系统。文件系统是由URI指定的，这里我们已使用hdfsURI来配置
HDFS为Hadoop默认系统。HDFS的守护程序通过该属性来确定HDFSnamenode的主机及端口。我们将在localhost默认的HDFS端口8020上运行namenode.
这样一来，HDFS客户端可以通过该属性得知namenode在哪里运行进而连接到它。  
第二项：dfs.replication,我们设为1，这样一来，HDFS就不回按默认设置将文件系统块复本设为3。在单独一个datanode上运行时，HDFS无法
将块复制到3个datanode上，所以会持续给出块复本不足的警告。设置这个属性后，问题就会解决。  
### 文件系统的基本操作
*从本地文件系统将一个文件复制到HDFS:*  
`
% hadoop fs -copyFromLocal input/docs/quangle.txt \ hdfs://loacalhost/user/tom/quangle.txt
`  
当然也可以简化命令格式以省略主机的URI并使用默认设置，省略hdfs://localhost,因为该项已在core-site.xml中指定
*也可以用相对路径home为上述的/user/tom/*  
`
% hadoop fs -copyFromLocal input/docs/quangle.txt quangle.txt
`  
*把文件复制回本地文件系统*
```
% hadoop fs -copyToLocal quangle.txt quangle.copy.txt
% md5 input/docs/quangle.txt quangle.copy.txt
```
若MD5键值相同，表明这个文件在HDFS之旅中得以幸存并保存完整
```
% hadoop fs -mkdir books
% hadoop fs -ls
Fond2 items
drw-xr-x - tom supergroup 0 2014-10-04 13:22 books
-rw-r--r-- 1 tom supergroup 119 20144-10-04 13:21 quangle.txt
```
第1列显示的是文件模式。第2列是文件的备份数。第3、4列是显示文件的所属用户和组别。第5列是文件的大小，以字节为单位,目录
为0.第6、7是文件的最后修改日期与时间。第8列是文件或目录的名称。  
