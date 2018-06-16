# Logstash + Kafka 实战应用

Logstash-1.51才开始内置Kafka插件，也就是说用之前的logstash版本是需要手动编译Kafka插件的，相信也很少人用了。建议使用2.3以上的logstash版本。

1、使用logstash向kafka写入一些数据

软件版本:

logstash 2.3.2

kafka\_2.11-0.10.0.1

logstash output 部分配置

output {

kafka {

```
workers =&gt; 2

bootstrap\_servers =&gt; "10.160.4.25:9092,10.160.4.26:9092,10.160.4.27:9092"

topic\_id =&gt; "xuexilog"
```

}

}

参数解释 :

workers：用于写入时的工作线程

bootstrap\_servers：指定可用的kafka broker实例列表

topic\_id：指定topic名称，可以在写入前手动在broker创建定义好分片数和副本数，也可以不提前创建，那么在logstash写入时会自动创建topic，分片数和副本数则默认为broker配置文件中设置的。

2、使用logstash消费一些数据，并写入到elasticsearch

软件版本:

logstash 2.3.2

elasticsearch-2.3.4

logstash 配置文件

input{

```
kafka {

    zk\_connect =&gt; "112.100.6.1:2181,112.100.6.2:2181,112.100.6.3:2181"

    group\_id =&gt; "logstash"

    topic\_id =&gt; "xuexilog"

    reset\_beginning =&gt; false

    consumer\_threads =&gt; 5

    decorate\_events =&gt; true
```

}

}

\# 这里group\_id 需要解释一下，在Kafka中，相同group的Consumer可以同时消费一个topic，不同group的Consumer工作则互不干扰。

\# 补充: 在同一个topic中的同一个partition同时只能由一个Consumer消费，当同一个topic同时需要有多个Consumer消费时，则可以创建更多的partition。

output {

```
if \[type\] == "nginxacclog" {

    elasticsearch {

        hosts =&gt; \["10.10.1.90:9200"\]

        index =&gt; "logstash-nginxacclog-%{+YYYY.MM.dd}"

        manage\_template =&gt; true

        flush\_size =&gt; 50000

        idle\_flush\_time =&gt; 10

        workers =&gt; 2
```

}

}

}

3、通过group\_id 查看当前详细的消费情况

bin/kafka-consumer-groups.sh --group logstash --describe --zookeeper 127.0.0.1:2181

wKiom1fiTL-xhYo5AABDZmsbids038.png

输出解释:

GROUP    TOPIC    PARTITION    CURRENT-OFFSET    LOG-END-OFFSET    LAG

消费者组    话题id    分区id    当前已消费的条数    总条数    未消费的条数

