# Logstash 处理 Mongod Log

logstash 可以处理各类日志



对于 Mongod 的 Log 来说，情况既简单又复杂



简单性在于 mongodb patterns 已经都定义好了，拿来就能用；复杂性在于，这样抓出来的信息几乎没有太大价值，无非是实现了一个日志存储的功能，谈不上分析，因为最重要的操作时长未能被抓取，而这个数值是分析慢操作的关键，然而 Mongod 日志在不同类别下message部分的格式完全不一样，操作耗时信息是可有可无的



Tip: grok 预定义的正则匹配可以参考 grok patterns ，mongo的日志规范可以参考 Mongodb Log，不同版本的格式也是不一样的



这里简单分享一下使用logstash处理 Mongod 日志的方法 ，相关的理论基础可以参考 grok



Tip: 当前的最新版本为 MongoDB 3.2.1 ，Logstash 2.2.1



概要

环境

CentOS release 6.6 \(Final\)

2.6.32-504.el6.x86\_64

logstash 2.1.1

elasticsearch-2.1.1

MongoDB version: 3.0.3

格式

从 MongoDB 3.0 开始，MongoDB 日志的格式包含了 severity level 和 component



&lt;timestamp&gt; &lt;severity&gt; &lt;component&gt; \[&lt;context&gt;\] &lt;message&gt;

Timestamp

默认是使用的 iso8601-local



Severity Levels

Level	Description

F	Fatal

E	Error

W	Warning

I	Informational, for Verbosity Level of 0

D	Debug, for All Verbosity Levels &gt; 0

Components

Item	Description

ACCESS	Messages related to access control, such as authentication

COMMAND	Messages related to database commands, such as count

CONTROL	Messages related to control activities, such as initialization

GEO	Messages related to the parsing of geospatial shapes, such as verifying the GeoJSON shapes

INDEX	Messages related to indexing operations, such as creating indexes

NETWORK	Messages related to network activities, such as accepting connections

QUERY	Messages related to queries, including query planner activities

REPL	Messages related to replica sets, such as initial sync and heartbeats

SHARDING	Messages related to sharding activities, such as the startup of the mongos

STORAGE	Messages related to storage activities, such as processes involved in the fsync command

JOURNAL	Messages related specifically to journaling activities

WRITE	Messages related to write operations, such as update commands

-	Messages not associated with a named component

Tip: 使用 db.getLogComponents\(\) 和 db.setLogLevel\(-1, "query"\) 来查看和设定日志级别



关注信息

从下面实例的格式中可以看到



2014-11-03T18:28:32.450-0500 I NETWORK  \[initandlisten\] waiting for connections on port 27017

2015-12-25T18:41:47.683+0800 I CONTROL  \[signalProcessingThread\] pid=37405 port=27017 64-bit host=mongodb-server

2015-12-25T18:51:43.858+0800 I QUERY    \[conn425412\] query local.oplog.rs query: { ts: { $gte: Timestamp 1450975902000\|10 } } planSummary: COLLSCAN cursorid:400229983803 ntoreturn:0 ntoskip:0 nscanned:0 nscannedObjects:102 keyUpdates:0 writeConflicts:0 numYields:11609 nreturned:101 reslen:18110 locks:{ Global: { acquireCount: { r: 11610 } }, MMAPV1Journal: { acquireCount: { r: 11611 } }, Database: { acquireCount: { r: 11610 }, acquireWaitCount: { r: 1 }, timeAcquiringMicros: { r: 165 } }, oplog: { acquireCount: { R: 11610 } } } 1211ms

2015-12-25T20:54:11.336+0800 I JOURNAL  \[journal writer\] old journal file will be removed: /var/lib/mongo/journal/j.\_177

2015-12-26T00:46:36.512+0800 I COMMAND  \[conn424487\] command feed\_test\_repo.$cmd command: geoNear { geoNear: "users", near: \[ 88.598884, 44.102866 \], query: {}, num: 30, maxDistance: 10 } keyUpdates:0 writeConflicts:0 numYields:399 reslen:37700 locks:{ Global: { acquireCount: { r: 400 } }, MMAPV1Journal: { acquireCount: { r: 400 } }, Database: { acquireCount: { r: 400 } }, Collection: { acquireCount: { R: 400 } } } 2584ms

2015-12-26T02:15:02.218+0800 I QUERY    \[conn429640\] assertion 13435 not master and slaveOk=false ns:feed\_test\_repo.notifications query:{ query: {}, orderby: { \_id: 1.0 } }

2015-12-26T13:50:20.755+0800 I REPL     \[ReplicationExecutor\] Member 192.168.100.123:27017 is now in state ARBITER

2015-12-29T01:45:40.781+0800 I STORAGE  \[FileAllocator\] allocating new datafile /var/lib/mongo/feed\_test\_repo.107, filling with zeroes...

参考



&lt;timestamp&gt; &lt;severity&gt; &lt;component&gt; \[&lt;context&gt;\] &lt;message&gt;

前四部分\(&lt;timestamp&gt; &lt;severity&gt; &lt;component&gt; \[&lt;context&gt;\]\)的内容相对固定

最后一部分 \(&lt;message&gt;\) 内部比较多变

我们比较关心操作时长，希望可以将这个信息收集进来，这个信息在最后一部分包含，有些内容包含，有些不包含

