# section4

一、前言

Logstash 是一个开源的数据收集引擎，它具有备实时数据传输能力。它可以统一过滤来自不同源的数据，并按照开发者的制定的规范输出到目的地。



顾名思义，Logstash 收集数据对象就是日志文件。由于日志文件来源多（如：系统日志、服务器 日志等），且内容杂乱，不便于人类进行观察。因此，我们可以使用 Logstash 对日志文件进行收集和统一过滤，变成可读性高的内容，方便开发者或运维人员观察，从而有效的分析系统/项目运行的性能，做好监控和预警的准备工作等。



二、安装

Logstash 依赖 JDK1.8 ，因此在安装之前请确保机器已经安装和配置好 JDK1.8。



Logstash 下载



tar -zxvf logstash-5.6.3.tar.gz -C /usr



cd logstash-5.6.3

三、组成结构

Logstash 通过管道进行运作，管道有两个必需的元素，输入和输出，还有一个可选的元素，过滤器。



输入插件从数据源获取数据，过滤器插件根据用户指定的数据格式修改数据，输出插件则将数据写入到目的地。如下图：



image



我们先来一个简单的案例：



bin/logstash -e 'input { stdin { } } output { stdout {} }'

启动 Logstash 后，再键入 Hello World，结果如下：



\[root@localhost logstash-5.6.3\]\# bin/logstash -e 'input { stdin { } } output { stdout {} }'

Sending Logstash's logs to /usr/logstash-5.6.3/logs which is now configured via log4j2.properties

\[2017-10-27T00:17:43,438\]\[INFO \]\[logstash.modules.scaffold\] Initializing module {:module\_name=&gt;"fb\_apache", :directory=&gt;"/usr/logstash-5.6.3/modules/fb\_apache/configuration"}

\[2017-10-27T00:17:43,440\]\[INFO \]\[logstash.modules.scaffold\] Initializing module {:module\_name=&gt;"netflow", :directory=&gt;"/usr/logstash-5.6.3/modules/netflow/configuration"}

\[2017-10-27T00:17:43,701\]\[INFO \]\[logstash.pipeline        \] Starting pipeline {"id"=&gt;"main", "pipeline.workers"=&gt;1, "pipeline.batch.size"=&gt;125, "pipeline.batch.delay"=&gt;5, "pipeline.max\_inflight"=&gt;125}

\[2017-10-27T00:17:43,744\]\[INFO \]\[logstash.pipeline        \] Pipeline main started

The stdin plugin is now waiting for input:

\[2017-10-27T00:17:43,805\]\[INFO \]\[logstash.agent           \] Successfully started Logstash API endpoint {:port=&gt;9600}

Hello World

2017-10-27T07:17:51.034Z localhost.localdomain Hello World

Hello World（输入）经过 Logstash 管道（过滤）变成：2017-10-27T07:17:51.034Z localhost.localdomain Hello World （输出）。



在生产环境中，Logstash 的管道要复杂很多，可能需要配置多个输入、过滤器和输出插件。



因此，需要一个配置文件管理输入、过滤器和输出相关的配置。配置文件内容格式如下：



\#　输入

input {

  ...

}



\# 过滤器

filter {

  ...

}



\# 输出

output {

  ...

}

根据自己的需求在对应的位置配置 输入插件、过滤器插件、输出插件 和 编码解码插件 即可。



四、插件用法

在使用插件之前，我们先了解一个概念：事件。



Logstash 每读取一次数据的行为叫做事件。



在 Logstach 目录中创建一个配置文件，名为 logstash.conf（名字任意）。



4.1 输入插件

输入插件允许一个特定的事件源可以读取到 Logstash 管道中，配置在 input {} 中，且可以设置多个。



修改配置文件：



input {

    \# 从文件读取日志信息

    file {

        path =&gt; "/var/log/messages"

        type =&gt; "system"

        start\_position =&gt; "beginning"

    }

}



\# filter {

\#

\# }



output {

    \# 标准输出

    stdout { codec =&gt; rubydebug }

}

其中，messages 为系统日志。



保存文件。键入：



bin/logstash -f logstash.conf

在控制台结果如下：



{

      "@version" =&gt; "1",

          "host" =&gt; "localhost.localdomain",

          "path" =&gt; "/var/log/messages",

    "@timestamp" =&gt; 2017-10-29T07:30:02.601Z,

       "message" =&gt; "Oct 29 00:30:01 localhost systemd: Starting Session 16 of user root.",

          "type" =&gt; "system"

}

......

4.2 输出插件

输出插件将事件数据发送到特定的目的地，配置在 output {} 中，且可以设置多个。



修改配置文件：



input {

    \# 从文件读取日志信息

    file {

        path =&gt; "/var/log/error.log"

        type =&gt; "error"

        start\_position =&gt; "beginning"

    }

    

}



\# filter {

\#

\# }



output {

    \# 输出到 elasticsearch

    elasticsearch {

        hosts =&gt; \["192.168.2.41:9200"\]

        index =&gt; "error-%{+YYYY.MM.dd}"

    }

}

其中，error.log 的内容格式如下：



2017-08-04 13:57:30.378 \[http-nio-8080-exec-1\] ERROR c.g.a.global.ResponseResultAdvice -设备数据为空

com.light.pay.common.exceptions.ValidationException: 设备数据为空

    at com.light.pay.common.validate.Check.isTrue\(Check.java:31\)

    at com.light.attendance.controllers.cloudApi.DevicePushController.deviceInfoPush\(DevicePushController.java:44\)

    at sun.reflect.NativeMethodAccessorImpl.invoke0\(Native Method\)

    at sun.reflect.NativeMethodAccessorImpl.invoke\(NativeMethodAccessorImpl.java:62\)

    at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run\(TaskThread.java:61\)

    at java.lang.Thread.run\(Thread.java:745\)

2017-08-04 13:57:44.495 \[http-nio-8080-exec-2\] ERROR c.g.a.global.ResponseResultAdvice -Failed to invoke remote method: pushData, provider: dubbo://192.168.2.100:20880/com.light.attendance.api.DevicePushApi?application=salary-custom&default.check=false&default.timeout=30000&dubbo=2.8.4&interface=com.light.attendance.api.DevicePushApi&methods=getAllDevices,getDeviceById,pushData&organization=com.light.attendance&ow

......

配置文件中使用 elasticsearch 输出插件。输出的日志信息将被保存到 Elasticsearch 中，索引名称为 index 参数设置的格式。



如果读者不了解 Elasticsearch 基础内容，可以查看本站 《Elasticsearch 基础入门》 文章或自行百度进行知识的补缺。



保存文件。键入：



bin/logstash -f logstash.conf

打开浏览器访问 http://192.168.2.41:9100 使用 head 插件查看 Elasticsearch 数据，结果如下图：



image



踩坑提醒：

file 输入插件默认使用 “\n” 判断日志中每行的边界位置。error.log 是笔者自己编辑的错误日志，之前由于在复制粘贴日志内容时，忘记在内容末尾换行，导致日志数据始终无法导入到 Elasticsearch 中。 在此，提醒各位读者这个关键点。



4.3 编码解码插件

编码解码插件本质是一种流过滤器，配合输入插件或输出插件使用。



从上图中，我们发现一个问题：Java 异常日志被拆分成单行事件记录到 Elasticsearch 中，这不符合开发者或运维人员的查看习惯。因此，我们需要对日志信息进行编码将多行事件转成单行事件记录起来。



我们需要配置 Multiline codec 插件，这个插件可以将多行日志信息合并成一行，作为一个事件处理。



Logstash 默认没有安装该插件，需要开发者自行安装。键入：



bin/logstash-plugin install logstash-codec-multiline

修改配置文件：



input {

    \# 从文件读取日志信息

    file {

        path =&gt; "/var/log/error.log"

        type =&gt; "error"

        start\_position =&gt; "beginning"

        \# 使用 multiline 插件

        codec =&gt; multiline {

            \# 通过正则表达式匹配，具体配置根据自身实际情况而定

            pattern =&gt; "^\d"

            negate =&gt; true

            what =&gt; "previous"

        }

    }



}



\# filter {

\#

\# }



output {

    \# 输出到 elasticsearch

    elasticsearch {

        hosts =&gt; \["192.168.2.41:9200"\]

        index =&gt; "error-%{+YYYY.MM.dd}"

    }

}

保存文件。键入：



bin/logstash -f logstash.conf

使用 head 插件查看 Elasticsearch 数据，结果如下图：



image



4.4 过滤器插件

过滤器插件位于 Logstash 管道的中间位置，对事件执行过滤处理，配置在 filter {}，且可以配置多个。



本次测试使用 grok 插件演示，grok 插件用于过滤杂乱的内容，将其结构化，增加可读性。



安装：



bin/logstash-plugin install logstash-filter-grok

修改配置文件：



input {

     stdin {}

}





filter {

     grok {

       match =&gt; { "message" =&gt; "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER

:duration}" }

     }

}





output {

     stdout {

        codec =&gt; "rubydebug"

     }

}

保存文件。键入：



bin/logstash -f logstash.conf

启动成功后，我们输入：



55.3.244.1 GET /index.html 15824 0.043

控制台返回：



\[root@localhost logstash-5.6.3\]\# bin/logstash -f logstash.conf 

Sending Logstash's logs to /root/logstash-5.6.3/logs which is now configured via log4j2.properties

\[2017-10-30T08:23:20,456\]\[INFO \]\[logstash.modules.scaffold\] Initializing module {:module\_name=&gt;"fb\_apache", :directory=&gt;"/root/logstash-5.6.3/modules/fb\_apache/configuration"}

\[2017-10-30T08:23:20,459\]\[INFO \]\[logstash.modules.scaffold\] Initializing module {:module\_name=&gt;"netflow", :directory=&gt;"/root/logstash-5.6.3/modules/netflow/configuration"}

\[2017-10-30T08:23:21,447\]\[INFO \]\[logstash.pipeline        \] Starting pipeline {"id"=&gt;"main", "pipeline.workers"=&gt;1, "pipeline.batch.size"=&gt;125, "pipeline.batch.delay"=&gt;5, "pipeline.max\_inflight"=&gt;125}

The stdin plugin is now waiting for input:

\[2017-10-30T08:23:21,516\]\[INFO \]\[logstash.pipeline        \] Pipeline main started

\[2017-10-30T08:23:21,573\]\[INFO \]\[logstash.agent           \] Successfully started Logstash API endpoint {:port=&gt;9600}

55.3.244.1 GET /index.html 15824 0.043

{

      "duration" =&gt; "0.043",

       "request" =&gt; "/index.html",

    "@timestamp" =&gt; 2017-10-30T15:23:23.912Z,

        "method" =&gt; "GET",

         "bytes" =&gt; "15824",

      "@version" =&gt; "1",

          "host" =&gt; "localhost.localdomain",

        "client" =&gt; "55.3.244.1",

       "message" =&gt; "55.3.244.1 GET /index.html 15824 0.043"

}

输入的内容被匹配到相应的名字中。



五、参考资料

https://www.elastic.co/guide/en/logstash/current/index.html 官方文档

