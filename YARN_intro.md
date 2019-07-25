# 关于YARN
YARN是Hadoop的集群资源管理系统。  

| 应用 | MapReduce | Spark | Tez | ... |
| :---: | :---: |:---: | :---: | :---: |
| 系统 | YARN | YARN | YARN | YARN |
| 存储 | HDFSandHBase | HDFSandHBase | HDFSandHBase | HDFSandHBase |
## 剖析YARN应用运行机制
![YARN](YARN.png)
步骤1: 要求他运行一个application master进程  
步骤2: 资源管理器找到一个能够在容器中启动application master的节点管理器  
步骤3: 可能向资源管理器请求更多的容器  
步骤4: 用于运行一个分布式计算。之后就是MapReduceYARN应用所做的事情。  
大多数重要的YARN应用使用某种的远程通信机制（例如Hadoop的RPC层）来向客户端传递状态更新和返回结果，但是这些通信机制都是
专属于各应用层。  
## YARN与MapReduce1相比

|MapReduce|YARN|
|:--- |:--- |
|Jobtracker|资源管理器、application、时间轴服务器|
|Tasktracker|节点管理器|
|Slot|容器|
YARN的很多设计是为了解决MapReduce1的局限性
### 可扩展性
如：当节点数达到4000，任务数到达40000时，MapReduce1会遇到可扩展性瓶颈，瓶颈源自于jobtracker必须同时管理作业和任务这样一个事实。
YARN利用其资源管理器和application master分离的架构有点克服这个局限性，可以扩展到面向将近10000个节点和100000个任务
### 可用性
由于YARN中的jobtrackder在资源管理器和application master之间进行了职责划分，高可用的服务随之成为一个分而治之问题：先为
资源管理器提供高可用性，再为YARN应用提供高可用性。
### 利用率
MapReduce1中，没个够tasktracker都配置有若干固定长度的slot，这些slot是静态分配的，在配置的时候就被划分为map slot和reduce slot.
一个map slot仅能用于运行一个map任务，同样，一个reduce slot仅能用于运行一个reduce任务。  
YARN中，一个节点管理器管理一个资源池，而不是指定的固定数目的slot.YARN上运行的MapReduce不会出现由于集群中仅有map slot可用导致reduce
任务必须等待的情况，而MapReduce1则会有这样的问题
### 多租户
YARN的最大优点在于向MapReduce以外的其他类型的分布式应用开放了Hadoop.MapReduce仅仅是许多YARN应用中的一个。
