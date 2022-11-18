#### 大数据软件性能优化

- SQL语句优化
- 数据倾斜处理，比如join的时候，某个key对应数据过多，会导致这个reduce长时间无法完成
- MapReduce、Spark代码优化
- 配置参数优化
- 大数据开源软件代码优化



#### Spark性能优化步骤

1. 性能测试，观察Spark性能特性和资源使用情况
2. 分析、寻找资源瓶颈
3. 分析系统架构、代码，发现资源利用关键所在
4. 代码、架构、基础设施调优，优化、平衡资源利用
5. 性能测试，观察系统性能特性是否达到优化目的



#### 案例分析

对下面的计算时序图，对Worker服务器上的资源进行监控，得到下面图

![img](https://static001.geekbang.org/resource/image/2b/d0/2bf9e431bbd543165588a111513567d0.png?wh=668*188)

![img](https://static001.geekbang.org/resource/image/7a/cc/7af885b0492aa68ffbe05bee7e04cdcc.png?wh=552*156)

![img](https://static001.geekbang.org/resource/image/2f/12/2f8f43d795247f575e953a027070d012.png?wh=540*152)



![img](https://static001.geekbang.org/resource/image/86/56/86cb0d0beea4178cbf5f7031fec7a956.png?wh=560*156)



![img](https://static001.geekbang.org/resource/image/09/33/093c888a13a58802413f9d1e2eafcb33.png?wh=544*152)





#### 案例1 任务文件初始化调优

一个迭代程序

![img](https://static001.geekbang.org/resource/image/6f/88/6fd436e3c6c11106cd7754792e78ee88.png?wh=670*206)

![img](https://static001.geekbang.org/resource/image/05/58/054abfc46ca040d3db8c441822a86558.png?wh=696*196)

- 分析CPU资源图发现，第一个Job比其他job多了一个stage，并且耗费了14s时间
- 分析代码发现，第一个job需要初始化一个空数组，所以产生了一个stage，但是耗时不合理
- 分析网络资源图发现，这段时间有网络开销
- 分析Spark运行日志，发现耗时原因是从Driver下载执行代码，但是代码只有17MB，传输不需要耗时这么多
- 继续分析代码，发现每个计算节点会启动多个Excutor进程计算，每个Excutor都会去下载Jar包，导致耗时变长
- 修改代码，同一个服务器上多个Executor下，由一个Executor下载代码到本地，其他进行copy到自己工作路径就好了



#### 案例2 Spark任务调度调优

![img](https://static001.geekbang.org/resource/image/49/38/498e4d3d7aa0c23b6fc5807eb87b7638.png?wh=466*154)

![img](https://static001.geekbang.org/resource/image/6e/64/6eb9b6f7ff05a9d521035898f830d964.png?wh=452*148)

![img](https://static001.geekbang.org/resource/image/b6/00/b6e5868d6af8ffbd3ddc99a0ad9e4b00.png?wh=466*152)

![img](https://static001.geekbang.org/resource/image/f8/5b/f82efd934ed0e1992cb7bb9460b9175b.png?wh=458*150)

- 根据资源图可以发现，在第一个job第二个阶段，只有一个机器的CPU很高，说明计算资源利用不均衡。
- 分析Spark运行日志和Spark源码，发现当有空闲计算资源Worker节点向Driver注册的时候，就会触发Spark的任务分配，分配使用轮询方式，所以为什么会导致不均衡呢？
- 继续分析日志发现，Worker节点注册时间有先后，第一个Worker注册后，如果需要执行的任务数小于Worker提供的计算单元数，就会出现一个Worker把所有任务领走的情况
- 增加配置项，只有注册的计算资源达到一定比例80%才开始分配任务，保证任务分配均衡
- 增加配置项，增加最大等待时间3s，防止一直达不到比例不开始分配资源



#### 案例3 Spark应用配置优化

![img](https://static001.geekbang.org/resource/image/b5/70/b5557db2bcfad01dc9d6e5506d77ea70.png?wh=568*160?wh=568*160)

- 通过资源图分析，服务器资源使用率一直在60%以下，无法充分利用资源
- 分析配置，发现Executor配置为120，一共四个Worker节点，每个节点只有30个任务，而服务器配置是48核的，所以最高也就30/48=62。5%
- 修改启动参数的Executor为48*4=192



#### 案例4 操作系统配置优化

太难了，不写了





#### 案例5 硬件优化

缺啥补啥