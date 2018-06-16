# flume介绍及原理总结  

一、flume简介

Flume是Cloudera提供的日志收集系统，Flume支持在日志系统中定制各类数据发送方，用于收集数据;同时，Flume提供对数据进行简单处理，并写到各种storage。Flume是一个分布式、可靠、和高可用的海量日志采集、聚合和传输的系统。

flume介绍及扩展开发心得一二 - labugao - 陈李平的博客

上图的Flume的Architecture，在Flume中，最重要的抽象是data flow\(数据流\)，data flow描述了数据从产生，传输、处理并最终写入目标的一条路径。在上图中，实线描述了data flow。



其中，Agent用于采集数据，agent是flume中产生数据流的地方，同时，agent会将产生的数据流传输到collector。对应的，collector用于对数据进行聚合，往往会产生一个更大的流。



　　Flume提供了从console\(控制台\)、RPC\(Thrift-RPC\)、text\(文件\)、tail\(UNIX tail\)、syslog\(syslog日志系统，支持TCP和UDP等2种模式\)，exec\(命令执行\)等数据源上收集数据的能力。同时，Flume的数据接受方，可以是console\(控制台\)、text\(文件\)、dfs\(HDFS文件\)、RPC\(Thrift-RPC\)和syslogTCP\(TCP syslog日志系统\)等。



　　其中，收集数据有2种主要工作模式，如下：



　　Push Sources：外部系统会主动地将数据推送到Flume中，如RPC、syslog。



　　Polling Sources：Flume到外部系统中获取数据，一般使用轮询的方式，如text和exec。



　　注意，在Flume中，agent和collector对应，而source和sink对应。Source和sink强调发送、接受方的特性\(如数据格式、编码等\)，而agent和collector关注功能。



　　Flume Master用于管理数据流的配置，如下图。



flume介绍及扩展开发心得一二 - labugao - 陈李平的博客

为了保证可扩展性，Flume采用了多Master的方式。为了保证配置数据的一致性，Flume引入了ZooKeeper，用于保存配置数据，ZooKeeper本身可保证配置数据的一致性和高可用，另外，在配置数据发生变化时，ZooKeeper可以通知Flume Master节点。



　　Flume Master间使用gossip协议同步数据。



　　下面简要分析Flume如何支持Reliability、Scalability、Manageability和Extensibility。



　　Reliability：Flume提供3中数据可靠性选项，包括End-to-end、Store on failure和Best effort。其中End-to-end使用了磁盘日志和接受端Ack的方式，保证Flume接受到的数据会最终到达目的。Store on failure在目的不可用的时候，数据会保持在本地硬盘。和End-to-end不同的是，如果是进程出现问题，Store on failure可能会丢失部分数据。Best effort不做任何QoS保证。



　　Scalability：Flume的3大组件：collector、master和storage tier都是可伸缩的。需要注意的是，Flume中对事件的处理不需要带状态，它的Scalability可以很容易实现。



　　Manageability：Flume利用ZooKeeper和gossip，保证配置数据的一致性、高可用。同时，多Master，保证Master可以管理大量的节点。



　　Extensibility：基于Java，用户可以为Flume添加各种新的功能，如通过继承Source，用户可以实现自己的数据接入方式，实现Sink的子类，用户可以将数据写往特定目标，同时，通过SinkDecorator，用户可以对数据进行一定的预处理。



 注：以上介绍来自：http://caibinbupt.iteye.com/blog/765960，更多了解请参考Flume主页：https://github.com/cloudera/flume/

二、为什么选择flume

目前可选的开源日志收集项目有如下这些：facebook的scribe，apache的chukwa，linkedin的kafka和cloudera的flume，注：flume正逐步迁移到apache下。其他项目的介绍课参考各项目主页:

scribe主页：https://github.com/facebook/scribe

chukwa主页：http://incubator.apache.org/chukwa/

kafka主页：http://sna-projects.com/kafka/

在此参考网友绘制的对比图表：

flume介绍及扩展开发心得一二 - labugao - 陈李平的博客

（图表来自：http://dongxicheng.org/search-engine/log-systems/）

从上图中可以看出flume作为开源的日志收集项目比较优秀，使用广泛，参考资料比较多。整体设计架构提供了强大的可扩展性和丰富的自带插件。

三、我们做了些什么

flume作为数据流平台中日志数据收集模块的核心组件，我们使用了他强大的收集和分流功能，在原有的基础上加上了分流配置的可管理功能。把日志的分流集中管理，有效的避免了flume原有分流方式的弊端：一种分流方式失败导致日志重复发送。日志的分流配置以及归档配置使用外部管理的方式，从而使系统的运维更方便。

日志存入hdfs的分流方式涉及到归档问题，flume本身提供了collectorSink支持日志数据存入hdfs，默认只支持按时间的归档方式，查看源码很容易发现flume也支持大小归档，甚至可以时间归档和大小归档同时使用。但是我们要求的更多，线上往往要求一个文件的大小不能超过64M，但归档的文件也不能太小，这就需要管理员根据产品高峰和低谷的日志量来分配归档规则，并且实现大小归档不能重置时间归档的触发器，因此这一需求需要改写flume本身提供的trigger。

flume中hdfs文件的归档靠两个地方实现，具体实现见RollSink.java,以下列举代码片段：

public void synchronousAppend\(Event e\) throws IOException,



      InterruptedException {



    Preconditions.checkState\(curSink != null,



        "Attempted to append when rollsink not open"\);



    if \(trigger.isTriggered\(\)\) {



      trigger.reset\(\);



      LOG.debug\("Rotate started by append... "\);



      rotate\(\);



      LOG.debug\("... rotate completed by append."\);



    }



    String tag = trigger.getTagger\(\).getTag\(\);



    e.set\(A\_ROLL\_TAG, tag.getBytes\(\)\);



    lock.readLock\(\).lock\(\);



    try {



      curSink.append\(e\);



      trigger.append\(e\);



      super.append\(e\);



    } finally {



      lock.readLock\(\).unlock\(\);



    }



  }



如上代码可以看到，每条日志过来都需要trigger判断是否达到归档条件。如果达到归档条件则惊醒归档，并且把该条日志写入新的文件中。个人觉得如上绿色和红色代码应该交换顺寻，触发归档的日志应该写入当前文档，而不是写入新创建的文档中，理由有两点：1、日志从产品处发送到flume虽然时间很短，但是触发归档的日志产生时间是大于归档时间的，理论上是属于上个归档规则之内的日志，例如：时间方式归档。

另外在RollSink内部有个线程每隔250毫秒执行一次如下归档判断：

 while \(!isInterrupted\(\)\) {



          // TODO there should probably be a lock on Roll sink but until we



          // handle



          // interruptions throughout the code, we cannot because this causes a



          // deadlock



          if \(trigger.isTriggered\(\)\) {



            trigger.reset\(\);



            LOG.debug\("Rotate started by triggerthread... "\);



            rotate\(\);



            LOG.debug\("Rotate stopped by triggerthread... "\);



           continue;



          }



          try {



            Clock.sleep\(checkLatencyMs\);



          } catch \(InterruptedException e\) {



            LOG.debug\("TriggerThread interrupted"\);



            doneLatch.countDown\(\);



            return;



          }



}



该线程用于保证当没有日志过来的时候，按时间归档的日志能够及时归档，归档的方式为关闭RollSink中curSink,重新open。

我们并没有采用上述flume原生的归档触发方式，因为我们系统中日志的分流采用聚合方式负责分流，一条日志可能分流到几种不同的storage,都是在一个sink中管理，因此hdfs的分流Sink需要负责多种tag的日志的归档，而这些文件的归档时间和大小各不相同，我们采用了在日志流中加入归档信号标志，当负责写hdfs的Sink接收到表示需要归档的event的时，提取出需要归档的hdfsWriter，作reopen操作。

四、问题及解决

在使用flume过程中遇到两个奇怪的问题：

1、flume使用lzo压缩方式存储到hdfs的配置，网上查了很多，都是介绍只要在flume-site.xml中加上如下配置：

  &lt;property&gt;



        &lt;name&gt;flume.collector.dfs.compress.codec&lt;/name&gt;



        &lt;value&gt;Lzop&lt;/value&gt;



        &lt;description&gt;Writes formatted data compressed in specified codec to dfs. Value is None, GzipCodec, DefaultCodec \(deflate\), BZip2Codec, or any other Codec Hadoop is aware of &lt;/description&gt;



  &lt;/property&gt;



我们也按网上介绍的加上了如此配置，始终不行，后来看了flume的源码才发现，flume系统中使用到的hadoop的Jar文件需要读取到如下配置：

&lt;property&gt;



     &lt;name&gt;io.compression.codecs&lt;/name&gt;



     &lt;value&gt;com.hadoop.compression.lzo.LzopCodec&lt;/value&gt;



    &lt;description&gt;lzop&lt;/description&gt;



&lt;/property&gt;



就是告诉flume，存储的hdfs支持lzop压缩格式的存储。当flume所在节点有部署hadoop的话，就不需要上述配置。不知道官网文档中哟没有介绍到这一点，网上找了很久没有提到这一点，希望对其他人有帮助。

2、flume连接的hdfs需要kerbores进行权限验证，根据网站介绍同样只需要加上如下配置：

&lt;property&gt;



    &lt;name&gt;flume.security.kerberos.principal&lt;/name&gt;



    &lt;value&gt;flume/local@DOMAIN&lt;/value&gt;



    &lt;description&gt;&lt;/description&gt;



&lt;/property&gt;



&lt;property&gt;



    &lt;name&gt;flume.security.kerberos.keytab&lt;/name&gt;



    &lt;value&gt;/home/ds/flume/flume.keytab&lt;/value&gt;



    &lt;description&gt;&lt;/description&gt;



&lt;/property&gt;



依然不行，后来查找原因，其实和上一点相同，flume中依赖hadoop的jar需要读取hadoop的配置，而flume不和hadoop部署在一个节点的时候，很多配置无法获取，所以需要另外再flume环境变量中加入hadoop的配置,需要把hadoop关于keberos配置参数写到fluem/conf/目录下core-default.xml。



