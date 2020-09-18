---
title: HDFS原理
date: 2020-05-04 20:31:33
categories: 读书笔记
tags:
- 笔记
- HDFS
---
#### Block的副本放置策略

* 第一个副本: 放置在上传文件的DataNode; 如果是集群外提交, 随机挑选一台 磁盘不太满, CPU不太忙的节点
* 第二个副本: 放置在跟第一个副本不同机架的节点上(容灾备份)
* 第三个副本: 尽量跟第二个副本相同机架(节约网络资源)
* 更多副本: 随机节点

<!--more-->
#### HDFS读流程
![HDFS read](/images/hdfs/hdfs-read.png)

1. 首先调用FileSystem对象的**open**方法, 获取的是一个DistributedFileSystem的实例
2. DistributedFileSystem通过**RPC**获得文件的第一批block的**locations**, 同一block按照重复数会返回多个locations, 这些locations按照hadoop拓扑结构排序, 距离客户端近的排在前面
3. 前两步会返回一个FSDataInputStream对象, 该对象会封装成 **DFSInputStream**对象,DFSInputStream可以方便的管理DataNode和NameNode数据流. 客户端调用read方法, DFSInputStream就会找出离客户端最近的datanode并连接datanode.
4. 数据从datanode源源不断的流向客户端
5. 如果第一个block块的数据读完了, 就会关闭指向第一个block块的datanode连接, 接着读取下一个block块. 这些操作对客户端透明, 从客户端角度来看只是读一个持续不断的流
6. 如果第一批block读完了, DFSInputStream就会去NameNode拿下一批blocks的location,继续读, 当所有block读完, 关闭所有流.

#### HDFS写流程
![HDFS write](/images/hdfs/hdfs-write.png)

1. 客户端通过调用DistributedFileSystem的create方法, 创建一个新的文件
2. DistributedFileSystem通过RPC调用NameNode,去创建一个没有blocks关联的新文件. 创建前, NameNode会做各种校验, 比如文件是否存在, 客户端有无权限去创建等. 如果通过, NameNode就会记录下新文件, 否则就会抛出IO异常.
3. 前两步结束后返回FSDataOutputStream的对象, 和读文件等时候相似, FSDataOutputStream被封装成DFSOutputStream.客户端开始写数据到DFSOutputStream, DFSOutputStream会把数据切成一个个小packet, 然后排成队列data queue.
4. DataStreamer会去处理接受data queue, DataStreamer把packet按队列输出到管道的第一个DataNode中, 第一个DataNode又把packet输出到第二个DataNode中, 以此类推.
5. DFSOutputStream还有一个队列叫 ack queue, 也是由packet组成, 等待DataNode的收到响应, 当pipeline中的所有DataNode都表示已经收到的时候, 这时ack queue才会把对应的packet包移除
6. 客户端完成写数据后, 调用close方法关闭写入流
7. DataStreamer把剩余的包都刷到pipeline里,然后等待ack消息, 收到最后一个ack后, 通知DataNode把文件标记为已完成.
8. 一个重要的参数: dfs.client.block.write.replace-datanode-on-failure.min-replication

#### HDFS文件权限
* 与Linux文件权限类似
    * r
    * w
    * x 权限x对于文件忽略, 对于文件夹表示是否允许访问其内容
* 如果Linux用户lxm使用hadoop命令创建一个文件, 那么这个文件在HDFS中owner就是 lxm
* HDFS权限的目的: 阻止好人做错事, 而不是阻止坏人做坏事. HDFS相信, 你告诉我你是谁, 我就认为你是谁.

#### HDFS安全模式
1. NameNode启动的时候, 首先将fsimage载入内存, 并执行edits中的各项操作
2. 一旦在内存中成功建立文件系统元数据的映射, 则创建一个新的fsimage文件(无需SecondaryNameNode)和一个空的编辑日志(即improgress edits log)
3. 此刻namenode运行在安全模式. 即 namenode的文件系统对于客户端是只读的
4. 在此阶段,NameNode会收集各个DataNode的报告, 当数据块达到最小副本数(dfs.namenode.safemode.replication.min)以上时, 会被认为是“安全”的, 在一定比例(dfs.namenode.safemode.threshold-pct)的数据块被确定为“安全”后,安全模式结束
5. 当检测到副本数不足的数据块时, 该块会被复制直到达到最小副本数, 系统中数据块的位置并不是NameNode维护的, 而是以块列表形式存储在DataNode中
