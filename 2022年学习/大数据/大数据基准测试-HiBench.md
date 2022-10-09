#### 大数据基准测试的应用

大数据基础测试的主要用途是对各种大数据产品进行测试，检验大数据产品在不同硬件平台，不同数据量，不同计算任务下的性能表现。

当然，也可以对自己的大数据平台进行测试，了解自己大数据平台的状况。



#### 应用例子

- Spark刚出来的时候，Intel也是用基础测试工具HiBench对Spark和MapReduce做了测试，发现Spark拥有很好的表现。立即和Spark开发者谈了合作，加入了Spark开发，推动了Spark的发展，后来加入Apache，风靡全球的时候，Intel也从中受益，在大数据领域保持持续的影响力。



#### HiBench

HiBench是intel推出的大数据基准测试工具，HiBench内置了若干主要的大数据计算程序作为基准测试的负载

- Sort，对数据进行排序
- WordCount，词频统计
- TERASort：对1TB数据进行排序
- Bayes分类：机器学习分类算法，用于数据分类和预测
- k-means聚类，对数据集合规律进行挖掘的算法
- 逻辑回归：数据进行预测和回归的算法
- SQL：包括全表扫描、group by 、join几种典型查询SQL
- PageRank：Web排序算法
- 此外还有十几种常见大数据计算程序，支持MapReduce、Spark、Storm等。



#### HiBench使用

1. 配置：配置要测试的数据量、大数据运行环境和路径信息等基本参数

2. 初始化数据：生成准备要计算的数据，直接运行脚本就可以自动生成配置大小的数据

   ```
   
   bin/workloads/micro/terasort/prepare/prepare.sh
   ```

   

3. 执行测试：运行对应的大数据计算程序。

   ```
   
   bin/workloads/micro/terasort/hadoop/run.sh
   bin/workloads/micro/terasort/spark/run.sh
   ```

   



![img](https://static001.geekbang.org/resource/image/96/9b/961e6cc96cb0beb649d96bd21ed62b9b.png?wh=1920*811)