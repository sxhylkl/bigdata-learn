## Spark Streaming

常见批处理方式：
- 单独处理每条记录，并在记录出现时立刻处理；
- 把多个记录组合为小批量任务，可以通过记录数量或者时间长度切分出来。spark streaming采用这种方式，核心概念是 **离散化流(DStream)**

与批处理不同，流处理应用需要保证不间断运行，需要的支持：
- checkpointing机制：把数据存到可靠文件系统（如HDFS）上
- 自动重启模式



转化操作：
- 无状态的操作
- 有状态的操作， 需要checkpointing机制来确保
    - 基于窗口的转化操作
    - UpdateStateByKey转化操作
