#### 概念

Flink即可以流处理又可以批处理，如果把从HDFS中读入的数据当做数据流看待，他就是批处理系统。



#### 原理

- 如果进行流计算，Flink会初始化流执行环境StreamExecutionEnvironment，利用这个执行环境构建数据流DataStream

  ```
  
  StreamExecutionEnvironment see = StreamExecutionEnvironment.getExecutionEnvironment();
  
  DataStream<WikipediaEditEvent> edits = see.addSource(new WikipediaEditsSource());
  ```

  

- 如果进行批处理计算，Flink会初始化批处理执行环境ExecutionEnvironment，利用这个执行环境构建数据集DataSet

  ```
  
  ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
  
  DataSet<String> text = env.readTextFile("/path/to/file");
  ```

  

- 然后在DataStream或者DataSet上执行各种数据转换操作，很像Spark处理。

- 不管是流处理还是批处理，Flink运行时的执行引擎是相同的，只是数据源不同。

- Flink处理实时数据流的方式跟Spark Streaming也很类似，也是将数据流分段后，一小批一小批的处理，但是Flink对于流处理的支持更加完善，可以对数据流执行window操作，将数据流切分到一个一个的window里，进行计算。

```

.timeWindow(Time.seconds(10))
```





#### Flink架构

Flink架构和Hadoop 1 或者Yarn类似

![img](https://static001.geekbang.org/resource/image/92/9f/92584744442b15d541a355eb7997029f.png?wh=645*480)



- JobManager：集群的管理者，client提交请求给JobManager后，检查集群中所有TaskManager的资源利用状况，如果有空闲TaskSlot（任务槽），就将计算任务分配给它执行
- TaskManager：任务执行者
- TaskSlot：具体计算任务