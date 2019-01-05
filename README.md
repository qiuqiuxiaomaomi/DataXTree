# DataXTree
阿里异构数据源离线同步技术研究


![](https://i.imgur.com/50UbiYi.png)

<pre>
介绍
 
         DataX是一个异构数据源离线同步工具，致力于实现包括关系型数据库(Mysql, Oracle),
      HDFS, Hive, ODPS, HBASE, FTP等各种异构数据源之间稳定高效的数据同步功能。
</pre>

<pre>
设计理念

      为了解决异构数据源同步问题，DataX将复杂的网状的同步链路变成了星型数据链路，DataX作
      为中间传输载体负责连接各种数据源。当需要接入一个新的数据源的时候，只需要将此数据源对
      接到DataX，便能跟已有的数据源做到无缝数据同步。
</pre>

![](https://i.imgur.com/J0q0j9p.png)

<pre>
     DataX本身作为离线数据同步框架，采用Framework + plugin架构构建。将数据源读取和写入
     抽象成为Reader/Writer插件，纳入到整个同步框架中。

     Reader：Reader为数据采集模块，负责采集数据源的数据，将数据发送给Framework。

     Writer： Writer为数据写入模块，负责不断向Framework取数据，并将数据写入到目的端。

     Framework：Framework用于连接reader和writer，作为两者的数据传输通道，并处理缓冲，
                流控，并发，数据转换等核心技术问题
</pre>

![](https://i.imgur.com/BNM4qsO.png)

<pre>
核心模块介绍：

      DataX完成单个数据同步的作业，我们称之为Job，DataX接受到一个Job之后，将启动一个进程
      来完成整个作业同步过程。DataX Job模块是单个作业的中枢管理节点，承担了数据清理、子任务
      切分(将单一作业计算转化为多个子Task)、TaskGroup管理等功能。

      DataXJob启动后，会根据不同的源端切分策略，将Job切分成多个小的Task(子任务)，以便于并
      发执行。Task便是DataX作业的最小单元，每一个Task都会负责一部分数据的同步工作。

      切分多个Task之后，DataX Job会调用Scheduler模块，根据配置的并发数据量，将拆分成的
      Task重新组合，组装成TaskGroup(任务组)。每一个TaskGroup负责以一定的并发运行完毕分配
      好的所有Task，默认单个任务组的并发数量为5。

      每一个Task都由TaskGroup负责启动，Task启动后，会固定启动Reader—>Channel—>Writer的
      线程来完成任务
</pre>