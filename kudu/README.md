## Kudu

Kudu：一个能对快速数据进行快速分析的Hadoop存储引擎（既能处理快速数据，也能快速分析，并且只需要一个单一的存储引擎）
- 允许用户尽可能快的扫描数据，达到在HDFS中扫描原始文件速度的两倍
- 允许用户尽可能快的执行随机读/写，响应时间大约为1ms
- 以高可用、容错、持久的方式实现以上目标

架构设计
- kudu的核心是一个基于表的存储引擎。不提供SQL，也不存在SQL重写和SQL执行计划
- 角色：master服务器、tablet服务器
    - master：管理Kudu集群的各种操作以及元数据，客户端与master交互来定义表或者获取表的属性或元数据【至少需要3台】
    - tablet：工作节点，执行所有与数据相关的操作【至少需要3台】
- 存储：列格式存储
- 两个主要使用场景：插入新数据、更新现有数据

#### 实时系统
基本的访问模式：
- 查看当前系统的情况，比如更新了版本之后系统最近的业务状态，评估是否正常
- 分析，研究数据的趋势，用最新的结果做了解和报告

对存储层的要求：
- 逐行插入，能够立即可读
- 低延迟随机读。能够高效的访问一小部分数据
- 快速分析和扫描。满足报表和即席查询，需要能够高效的扫描大规模数据
- 更新。上下文信息会变化。

使用传统的存储引擎：
- 迭代1：HDFS。
    - 比如一个spark streaming每10s运行一次生成一个小批量数据存入HDFS【TODO：这个写是写入新文件还是追加？性能怎么样？】
    
![image](img/使用HDFS的实时数据流.jpg)    

- 迭代2：HDFS+compaction
    - 存在的问题以及复杂性
        - 需要在一个分区不在活跃时进行compaction过程，并且不能把结果覆盖到同一分区中。写入一个新的位置并通过HDFS命令交换新老位置，还不太能保证数据的一致性
        - HDFS没有主键概念，还需要一个可去重的机制
        - 有延迟的到达的数据可能不会落到当天的compaction分区内

![image](img/使用了HDFS+compaction的实时数据流.png)

- 迭代3：HBase+HDFS
    - HBase和Cassandra都属于big table家族，致命弱点是不能提供HDFS和Parquet的快速分析和扫描
    - 复杂，开发、维护、操作成本都很高，实际上很少用这种方案
    
![image](img/使用了HBase和HDFS的实时数据流.png)


分开来看，不使用kudu通过Hadoop生态也可以拥有上面的要求，其系统架构如下，不足是很复杂且难以维护

![iamge](img/实时数据流.png)


#### 学习资料
- [《Kudu:构建高性能实时数据分析存储系统》](https://github.com/kudu-book/getting-started-kudu)