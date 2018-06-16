# section6

Logstash 是一个开源的，灵活的，数据收集、加工和传输管道，用来进行有效地处理连续增长的日志，事件信息和非结构数据，然后可以选择分发到不同形式的输出，包括Elasticsearch



Logstash is a flexible, open source, data collection, enrichment, and transport pipeline designed to efficiently process a growing list of log, event, and unstructured data sources for distribution into a variety of outputs, including Elasticsearch.



Logstash 是ELK的核心组件之一，产品的详细信息可以参考 官网 ，下面分享一下 Logstash 基础操作，详细可以参阅 官方文档



Tip: 当前的最新版本为 Logstash 2.1.1



概要

依赖

Logstash requires Java 7 or later

Logstash requires Java 7 or later. Use the official Oracle distribution or an open-source distribution such as OpenJDK .



检查java版本

\[root@h102 ELK\]\# java -version

java version "1.7.0\_65"

OpenJDK Runtime Environment \(rhel-2.5.1.2.el6\_5-x86\_64 u65-b17\)

OpenJDK 64-Bit Server VM \(build 24.65-b04, mixed mode\)

\[root@h102 ELK\]\# 

符合要求



安装方法

可以参考 Package Repositories



基础测试

\[root@h102 ~\]\# /opt/logstash/bin/logstash -e 'input { stdin { } } output { stdout {} }'

Settings: Default filter workers: 1

Logstash startup completed

hello world

2015-12-23T03:26:40.336Z h102.temp hello world

abc

2015-12-23T03:27:03.461Z h102.temp abc

123

2015-12-23T03:27:46.412Z h102.temp 123

Logstash shutdown completed

\[root@h102 ~\]\# 

使用 -e 可以直接在命令中指定配置



使用 CTRL-D 进行终止



高级Logstash管道

大多数情况下Logstash有不止一个输入与输出，在配置更为复杂的情况下使用配置文件进行行为设定



使用 -f /path/to/conf 的方式指定配置文件



配置文件里有两个必要的定义 input 和 output ，还有一个可选的定义 filter ，input 用来指定数据来源，filter 用来进行数据处理 ，output 用来指定存储方式



basic\_logstash\_pipeline.png



现在读取Apache web日志，分析后写到Elasticsearch中



\[root@h102 logstash\]\# ls

logstash-tutorial.log

\[root@h102 logstash\]\# head -n 3 logstash-tutorial.log

83.149.9.216 - - \[04/Jan/2015:05:13:42 +0000\] "GET /presentations/logstash-monitorama-2013/images/kibana-search.png HTTP/1.1" 200 203023 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 \(Macintosh; Intel Mac OS X 10\_9\_1\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/32.0.1700.77 Safari/537.36"

83.149.9.216 - - \[04/Jan/2015:05:13:42 +0000\] "GET /presentations/logstash-monitorama-2013/images/kibana-dashboard3.png HTTP/1.1" 200 171717 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 \(Macintosh; Intel Mac OS X 10\_9\_1\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/32.0.1700.77 Safari/537.36"

83.149.9.216 - - \[04/Jan/2015:05:13:44 +0000\] "GET /presentations/logstash-monitorama-2013/plugin/highlight/highlight.js HTTP/1.1" 200 26185 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 \(Macintosh; Intel Mac OS X 10\_9\_1\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/32.0.1700.77 Safari/537.36"

\[root@h102 logstash\]\# vim first-pipeline.conf

\[root@h102 logstash\]\# cat first-pipeline.conf 

input {

    file {

        path =&gt; "/root/logstash/logstash-tutorial.log"

        start\_position =&gt; beginning 

    }

}



filter {

    grok {

        match =&gt; { "message" =&gt; "%{COMBINEDAPACHELOG}"}

    }

    geoip {

    	source =&gt; "clientip"

    }

}

output {

    elasticsearch {

	hosts =&gt; "localhost:9200"

    }

    stdout {}

}

\[root@h102 logstash\]\# /opt/logstash/bin/logstash -f first-pipeline.conf  -t

Configuration OK

\[root@h102 logstash\]\#

-t 可以进行配置检查



start\_position 代表从头开始读数据



grok geoip 是两个过滤插件



执行操作



\[root@h102 logstash\]\# /opt/logstash/bin/logstash -f first-pipeline.conf  

Settings: Default filter workers: 1

Logstash startup completed

2015-12-23T13:45:19.034Z h102.temp 83.149.9.216 - - \[04/Jan/2015:05:13:42 +0000\] "GET /presentations/logstash-monitorama-2013/images/kibana-search.png HTTP/1.1" 200 203023 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 \(Macintosh; Intel Mac OS X 10\_9\_1\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/32.0.1700.77 Safari/537.36"

2015-12-23T13:45:19.037Z h102.temp 83.149.9.216 - - \[04/Jan/2015:05:13:42 +0000\] "GET /presentations/logstash-monitorama-2013/images/kibana-dashboard3.png HTTP/1.1" 200 171717 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 \(Macintosh; Intel Mac OS X 10\_9\_1\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/32.0.1700.77 Safari/537.36"

2015-12-23T13:45:19.037Z h102.temp 83.149.9.216 - - \[04/Jan/2015:05:13:44 +0000\] "GET /presentations/logstash-monitorama-2013/plugin/highlight/highlight.js HTTP/1.1" 200 26185 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 \(Macintosh; Intel Mac OS X 10\_9\_1\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/32.0.1700.77 Safari/537.36"

...

...

elasticsearch中检索

使用下面的方式进行检索



查返回状态为 404 和 304的



\[root@h102 ~\]\# curl -XGET 'localhost:9200/logstash-2015.12.23/\_search?q=response=404'

{"took":3,"timed\_out":false,"\_shards":{"total":5,"successful":5,"failed":0},"hits":{"total":2,"max\_score":1.5351382,"hits":\[{"\_index":"logstash-2015.12.23","\_type":"logs","\_id":"AVHPFktn70zKhyBEHGid","\_score":1.5351382,"\_source":{"message":"66.249.73.185 - - \[04/Jan/2015:05:22:13 +0000\] \"GET /doc/index.html?org/elasticsearch/action/search/SearchResponse.html HTTP/1.1\" 404 294 \"-\" \"Mozilla/5.0 \(compatible; Googlebot/2.1; +http://www.google.com/bot.html\)\"","@version":"1","@timestamp":"2015-12-23T13:45:22.565Z","host":"h102.temp","path":"/root/logstash/logstash-tutorial.log","clientip":"66.249.73.185","ident":"-","auth":"-","timestamp":"04/Jan/2015:05:22:13 +0000","verb":"GET","request":"/doc/index.html?org/elasticsearch/action/search/SearchResponse.html","httpversion":"1.1","response":"404","bytes":"294","referrer":"\"-\"","agent":"\"Mozilla/5.0 \(compatible; Googlebot/2.1; +http://www.google.com/bot.html\)\"","geoip":{"ip":"66.249.73.185","country\_code2":"US","country\_code3":"USA","country\_name":"United States","continent\_code":"NA","region\_name":"CA","city\_name":"Mountain View","latitude":37.385999999999996,"longitude":-122.0838,"dma\_code":807,"area\_code":650,"timezone":"America/Los\_Angeles","real\_region\_name":"California","location":\[-122.0838,37.385999999999996\]}}},{"\_index":"logstash-2015.12.23","\_type":"logs","\_id":"AVHPFktm70zKhyBEHGhn","\_score":1.4070371,"\_source":{"message":"83.149.9.216 - - \[04/Jan/2015:05:13:45 +0000\] \"GET /presentations/logstash-monitorama-2013/images/frontend-response-codes.png HTTP/1.1\" 200 52878 \"http://semicomplete.com/presentations/logstash-monitorama-2013/\" \"Mozilla/5.0 \(Macintosh; Intel Mac OS X 10\_9\_1\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/32.0.1700.77 Safari/537.36\"","@version":"1","@timestamp":"2015-12-23T13:45:19.047Z","host":"h102.temp","path":"/root/logstash/logstash-tutorial.log","clientip":"83.149.9.216","ident":"-","auth":"-","timestamp":"04/Jan/2015:05:13:45 +0000","verb":"GET","request":"/presentations/logstash-monitorama-2013/images/frontend-response-codes.png","httpversion":"1.1","response":"200","bytes":"52878","referrer":"\"http://semicomplete.com/presentations/logstash-monitorama-2013/\"","agent":"\"Mozilla/5.0 \(Macintosh; Intel Mac OS X 10\_9\_1\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/32.0.1700.77 Safari/537.36\"","geoip":{"ip":"83.149.9.216","country\_code2":"RU","country\_code3":"RUS","country\_name":"Russian Federation","continent\_code":"EU","region\_name":"48","city\_name":"Moscow","latitude":55.75219999999999,"longitude":37.6156,"timezone":"Europe/Moscow","real\_region\_name":"Moscow City","location":\[37.6156,55.75219999999999\]}}}\]}}\[root@h102 ~\]\# 

\[root@h102 ~\]\# 

\[root@h102 ~\]\# 

\[root@h102 ~\]\# curl -XGET 'localhost:9200/logstash-2015.12.23/\_search?q=response=304&pretty'

{

  "took" : 6,

  "timed\_out" : false,

  "\_shards" : {

    "total" : 5,

    "successful" : 5,

    "failed" : 0

  },

  "hits" : {

    "total" : 2,

    "max\_score" : 1.570033,

    "hits" : \[ {

      "\_index" : "logstash-2015.12.23",

      "\_type" : "logs",

      "\_id" : "AVHPFktm70zKhyBEHGhn",

      "\_score" : 1.570033,

      "\_source":{"message":"83.149.9.216 - - \[04/Jan/2015:05:13:45 +0000\] \"GET /presentations/logstash-monitorama-2013/images/frontend-response-codes.png HTTP/1.1\" 200 52878 \"http://semicomplete.com/presentations/logstash-monitorama-2013/\" \"Mozilla/5.0 \(Macintosh; Intel Mac OS X 10\_9\_1\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/32.0.1700.77 Safari/537.36\"","@version":"1","@timestamp":"2015-12-23T13:45:19.047Z","host":"h102.temp","path":"/root/logstash/logstash-tutorial.log","clientip":"83.149.9.216","ident":"-","auth":"-","timestamp":"04/Jan/2015:05:13:45 +0000","verb":"GET","request":"/presentations/logstash-monitorama-2013/images/frontend-response-codes.png","httpversion":"1.1","response":"200","bytes":"52878","referrer":"\"http://semicomplete.com/presentations/logstash-monitorama-2013/\"","agent":"\"Mozilla/5.0 \(Macintosh; Intel Mac OS X 10\_9\_1\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/32.0.1700.77 Safari/537.36\"","geoip":{"ip":"83.149.9.216","country\_code2":"RU","country\_code3":"RUS","country\_name":"Russian Federation","continent\_code":"EU","region\_name":"48","city\_name":"Moscow","latitude":55.75219999999999,"longitude":37.6156,"timezone":"Europe/Moscow","real\_region\_name":"Moscow City","location":\[37.6156,55.75219999999999\]}}

    }, {

      "\_index" : "logstash-2015.12.23",

      "\_type" : "logs",

      "\_id" : "AVHPFlOg70zKhyBEHGi0",

      "\_score" : 1.570033,

      "\_source":{"message":"218.30.103.62 - - \[04/Jan/2015:05:27:36 +0000\] \"GET /projects/xdotool/xdotool.xhtml HTTP/1.1\" 304 - \"-\" \"Sogou web spider/4.0\(+http://www.sogou.com/docs/help/webmasters.htm\#07\)\"","@version":"1","@timestamp":"2015-12-23T13:45:22.923Z","host":"h102.temp","path":"/root/logstash/logstash-tutorial.log","clientip":"218.30.103.62","ident":"-","auth":"-","timestamp":"04/Jan/2015:05:27:36 +0000","verb":"GET","request":"/projects/xdotool/xdotool.xhtml","httpversion":"1.1","response":"304","referrer":"\"-\"","agent":"\"Sogou web spider/4.0\(+http://www.sogou.com/docs/help/webmasters.htm\#07\)\"","geoip":{"ip":"218.30.103.62","country\_code2":"CN","country\_code3":"CHN","country\_name":"China","continent\_code":"AS","region\_name":"22","city\_name":"Beijing","latitude":39.9289,"longitude":116.38830000000002,"timezone":"Asia/Harbin","real\_region\_name":"Beijing","location":\[116.38830000000002,39.9289\]}}

    } \]

  }

}

\[root@h102 ~\]\# 

加 pretty 参数，可以使输出更清晰



查来自 Buffalo 的



\[root@h102 ~\]\# curl -XGET 'localhost:9200/logstash-2015.12.23/\_search?q=geoip.city\_name=Buffalo&pretty'

{

  "took" : 4,

  "timed\_out" : false,

  "\_shards" : {

    "total" : 5,

    "successful" : 5,

    "failed" : 0

  },

  "hits" : {

    "total" : 1,

    "max\_score" : 1.0520113,

    "hits" : \[ {

      "\_index" : "logstash-2015.12.23",

      "\_type" : "logs",

      "\_id" : "AVHPFlOg70zKhyBEHGi1",

      "\_score" : 1.0520113,

      "\_source":{"message":"108.174.55.234 - - \[04/Jan/2015:05:27:45 +0000\] \"GET /?flav=rss20 HTTP/1.1\" 200 29941 \"-\" \"-\"","@version":"1","@timestamp":"2015-12-23T13:45:22.929Z","host":"h102.temp","path":"/root/logstash/logstash-tutorial.log","clientip":"108.174.55.234","ident":"-","auth":"-","timestamp":"04/Jan/2015:05:27:45 +0000","verb":"GET","request":"/?flav=rss20","httpversion":"1.1","response":"200","bytes":"29941","referrer":"\"-\"","agent":"\"-\"","geoip":{"ip":"108.174.55.234","country\_code2":"US","country\_code3":"USA","country\_name":"United States","continent\_code":"NA","region\_name":"NY","city\_name":"Buffalo","postal\_code":"14221","latitude":42.9864,"longitude":-78.7279,"dma\_code":514,"area\_code":716,"timezone":"America/New\_York","real\_region\_name":"New York","location":\[-78.7279,42.9864\]}}

    } \]

  }

}

\[root@h102 ~\]\# 

Filebeat

Filebeat 是一个轻量友好的工具，用来从目标服务器中收集文本日志然后然后转发给 Logstash 实例进行处理，其实就是一个 Logstash 的轻量前端文本收集代理



The filebeat client is a lightweight, resource-friendly tool that collects logs from files on the server and forwards these logs to your Logstash instance for processing



配置和启动filebeat



\[root@h102 etc\]\# grep -v "\#" /etc/filebeat/filebeat.yml  \| grep -v "^$"

filebeat:

  prospectors:

    -

      paths:

        - /var/log/\*.log

        - /var/log/messages\*

      input\_type: log

  registry\_file: /var/lib/filebeat/registry

output:

  logstash:

    hosts: \["localhost:5044"\]

shipper:

logging:

  files:

\[root@h102 etc\]\# /etc/init.d/filebeat start 

Starting filebeat:                                         \[  OK  \]

\[root@h102 etc\]\# /etc/init.d/filebeat status

filebeat-god \(pid  2770\) is running...

\[root@h102 etc\]\# ps faux \| grep filebeat

root      2787  0.0  0.0 103256   828 pts/1    S+   15:53   0:00          \\_ grep filebeat

root      2770  0.0  0.0  11388   232 pts/1    Sl   15:52   0:00 filebeat-god -r / -n -p /var/run/filebeat.pid -- /usr/bin/filebeat -c /etc/filebeat/filebeat.yml

root      2771  0.5  0.7 356380 13944 pts/1    Sl   15:52   0:00  \\_ /usr/bin/filebeat -c /etc/filebeat/filebeat.yml

\[root@h102 etc\]\# 

配置logstash并且运行



\[root@h102 etc\]\# cat  logstash-filebeat-es-simple.conf

input {

	stdin{}

	beats{port =&gt; 5044}

}

output {

	elasticsearch {

		hosts=&gt;"localhost:9200"

		index=&gt;"%{\[@metadata\]\[beat\]}-%{+YYYY.MM.dd}"

		document\_type =&gt; "%{\[@metadata\]\[type\]}"

	}

	stdout {codec=&gt;rubydebug}

}

\[root@h102 etc\]\# 

\[root@h102 etc\]\# /opt/logstash/bin/logstash -f logstash-filebeat-es-simple.conf 

Settings: Default filter workers: 1

Logstash startup completed

{

       "message" =&gt; "Dec 25 00:37:20 h102 init: tty \(/dev/tty2\) main process \(2300\) killed by TERM signal",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2016-01-05T07:52:48.994Z",

          "beat" =&gt; {

        "hostname" =&gt; "h102.temp",

            "name" =&gt; "h102.temp"

    },

         "count" =&gt; 1,

        "fields" =&gt; nil,

    "input\_type" =&gt; "log",

        "offset" =&gt; 247376,

        "source" =&gt; "/var/log/messages-20151230",

          "type" =&gt; "log",

          "host" =&gt; "h102.temp"

}

...

...

logstash的配置中加入了 stdout {codec=&gt;rubydebug} 是为了方便在终端监视信息\(在实际应用中完全没有必要\)，经过一番刷屏，最终停了下来



数据导入之前es里是这样的



\[root@h102 etc\]\# curl localhost:9200/\_cat/indices?v

health status index               pri rep docs.count docs.deleted store.size pri.store.size 

yellow open   filebeat-2015.12.24   5   1       3182            0        1mb            1mb 

yellow open   logstash-2015.12.23   5   1        100            0    235.8kb        235.8kb 

yellow open   logstash-2015.12.22   5   1         41            0    126.5kb        126.5kb 

yellow open   .kibana               1   1         94            0    102.3kb        102.3kb 

\[root@h102 etc\]\#

导入之后是这样的



\[root@h102 ~\]\# curl localhost:9200/\_cat/indices?v

health status index               pri rep docs.count docs.deleted store.size pri.store.size 

yellow open   filebeat-2015.12.24   5   1       3182            0        1mb            1mb 

yellow open   logstash-2015.12.23   5   1        100            0    235.8kb        235.8kb 

yellow open   logstash-2015.12.22   5   1         41            0    126.5kb        126.5kb 

yellow open   filebeat-2016.01.05   5   1       4182            0      1.3mb          1.3mb 

yellow open   .kibana               1   1         94            0    102.3kb        102.3kb 

\[root@h102 ~\]\# 

多了一个 filebeat-2016.01.05



查看数据



\[root@h102 ~\]\# curl -XGET 'localhost:9200/filebeat-2016.01.05/\_search?q=message=2935&pretty'

{

  "took" : 9,

  "timed\_out" : false,

  "\_shards" : {

    "total" : 5,

    "successful" : 5,

    "failed" : 0

  },

  "hits" : {

    "total" : 1,

    "max\_score" : 2.3564386,

    "hits" : \[ {

      "\_index" : "filebeat-2016.01.05",

      "\_type" : "log",

      "\_id" : "AVIQ3fOb0svkz\_zfzuMm",

      "\_score" : 2.3564386,

      "\_source":{"message":"Jan  5 16:18:37 h102 dhclient\[1624\]: bound to 192.168.1.117 -- renewal in 2935 seconds.","@version":"1","@timestamp":"2016-01-05T08:18:39.119Z","beat":{"hostname":"h102.temp","name":"h102.temp"},"count":1,"fields":null,"input\_type":"log","offset":166773,"source":"/var/log/messages","type":"log","host":"h102.temp"}

    } \]

  }

}

\[root@h102 ~\]\#

关闭步骤

关闭一个正在运行的 logstash 包含以下三步



停止所有的 input, filter 和 output 插件

完成所有的正在处理的事件

停止 Logstash 进程

使用 --allow-unsafe-shutdown 开启 Logstash 可以在中途强制关闭 Logstash ，会丢失数据



详细可以参考 Stalled Shutdown Detection



流水线模型

当前的 Logstash 是这样的处理模型



input threads \| filter worker threads \| output worker

deploy\_2.png



Filter 是可选项，如果没关于 Filter 的定义 ,就是如下模型



input threads \| output worker

deploy\_1.png



处理系统日志

可以在配置中加入判断与处理逻辑



\[root@h102 etc\]\# vim logstash-syslog.conf

\[root@h102 etc\]\# cat logstash-syslog.conf 

input {

  tcp {

    port =&gt; 5000

    type =&gt; syslog

  }

  udp {

    port =&gt; 5000

    type =&gt; syslog

  }

}



filter {

  if \[type\] == "syslog" {

    grok {

      match =&gt; { "message" =&gt; "%{SYSLOGTIMESTAMP:syslog\_timestamp} %{SYSLOGHOST:syslog\_hostname} %{DATA:syslog\_program}\(?:\\[%{POSINT:syslog\_pid}\\]\)?: %{GREEDYDATA:syslog\_message}" }

      add\_field =&gt; \[ "received\_at", "%{@timestamp}" \]

      add\_field =&gt; \[ "received\_from", "%{host}" \]

    }

    syslog\_pri { }

    date {

      match =&gt; \[ "syslog\_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" \]

    }

  }

}



output {

  elasticsearch { hosts =&gt; \["localhost:9200"\] }

  stdout { codec =&gt; rubydebug }

}

\[root@h102 etc\]\# /opt/logstash/bin/logstash -f logstash-syslog.conf  -t 

Configuration OK

\[root@h102 etc\]\#

打开本地的 tcp udp 5000端口

标记类型为 syslog

filter中作判断如果类型是 syslog

拆分解析信息

添加 received\_at received\_from 字段

使用 syslog\_pri { } 来处理

定义 syslog\_timestamp 的格式

输出到ES

以 rubydebug 的格式输出到终端

启动 Logstash



\[root@h102 etc\]\# /opt/logstash/bin/logstash -f logstash-syslog.conf  

Settings: Default filter workers: 1

Logstash startup completed

...

...

尝试手动连接本地 5000 端口，然后输入一些内容



\[root@h102 ~\]\# netstat  -ant \| grep 5000

tcp        0      0 :::5000                     :::\*                        LISTEN      

tcp        0      0 ::1:44814                   ::1:5000                    ESTABLISHED 

tcp        0      0 ::1:5000                    ::1:44814                   ESTABLISHED 

\[root@h102 ~\]\# telnet localhost 5000

Trying ::1...

Connected to localhost.

Escape character is '^\]'.

Dec 23 12:11:43 louis postfix/smtpd\[31499\]: connect from unknown\[95.75.93.154\]

Dec 23 14:42:56 louis named\[16000\]: client 199.48.164.7\#64817: query \(cache\) 'amsterdamboothuren.com/MX/IN' denied

...

...

发现这边的终端有输出



\[root@h102 etc\]\# /opt/logstash/bin/logstash -f logstash-syslog.conf  

Settings: Default filter workers: 1

Logstash startup completed

{

                 "message" =&gt; "Dec 23 12:11:43 louis postfix/smtpd\[31499\]: connect from unknown\[95.75.93.154\]\r",

                "@version" =&gt; "1",

              "@timestamp" =&gt; "2016-12-23T04:11:43.000Z",

                    "host" =&gt; "0:0:0:0:0:0:0:1",

                    "port" =&gt; 45093,

                    "type" =&gt; "syslog",

        "syslog\_timestamp" =&gt; "Dec 23 12:11:43",

         "syslog\_hostname" =&gt; "louis",

          "syslog\_program" =&gt; "postfix/smtpd",

              "syslog\_pid" =&gt; "31499",

          "syslog\_message" =&gt; "connect from unknown\[95.75.93.154\]\r",

             "received\_at" =&gt; "2016-01-05T12:22:55.674Z",

           "received\_from" =&gt; "0:0:0:0:0:0:0:1",

    "syslog\_severity\_code" =&gt; 5,

    "syslog\_facility\_code" =&gt; 1,

         "syslog\_facility" =&gt; "user-level",

         "syslog\_severity" =&gt; "notice"

}

{

                 "message" =&gt; "Dec 23 14:42:56 louis named\[16000\]: client 199.48.164.7\#64817: query \(cache\) 'amsterdamboothuren.com/MX/IN' denied\r",

                "@version" =&gt; "1",

              "@timestamp" =&gt; "2016-12-23T06:42:56.000Z",

                    "host" =&gt; "0:0:0:0:0:0:0:1",

                    "port" =&gt; 45093,

                    "type" =&gt; "syslog",

        "syslog\_timestamp" =&gt; "Dec 23 14:42:56",

         "syslog\_hostname" =&gt; "louis",

          "syslog\_program" =&gt; "named",

              "syslog\_pid" =&gt; "16000",

          "syslog\_message" =&gt; "client 199.48.164.7\#64817: query \(cache\) 'amsterdamboothuren.com/MX/IN' denied\r",

             "received\_at" =&gt; "2016-01-05T12:23:22.809Z",

           "received\_from" =&gt; "0:0:0:0:0:0:0:1",

    "syslog\_severity\_code" =&gt; 5,

    "syslog\_facility\_code" =&gt; 1,

         "syslog\_facility" =&gt; "user-level",

         "syslog\_severity" =&gt; "notice"

}

...

...

ES里也有了数据



\[root@h102 etc\]\# curl -XGET 'localhost:9200/logstash-2016.12.23/\_search?q=message=louis&pretty'

{

  "took" : 5,

  "timed\_out" : false,

  "\_shards" : {

    "total" : 5,

    "successful" : 5,

    "failed" : 0

  },

  "hits" : {

    "total" : 2,

    "max\_score" : 0.06365098,

    "hits" : \[ {

      "\_index" : "logstash-2016.12.23",

      "\_type" : "syslog",

      "\_id" : "AVIRvXxq0svkz\_zfzuOP",

      "\_score" : 0.06365098,

      "\_source":{"message":"Dec 23 12:11:43 louis postfix/smtpd\[31499\]: connect from unknown\[95.75.93.154\]\r","@version":"1","@timestamp":"2016-12-23T04:11:43.000Z","host":"0:0:0:0:0:0:0:1","port":45093,"type":"syslog","syslog\_timestamp":"Dec 23 12:11:43","syslog\_hostname":"louis","syslog\_program":"postfix/smtpd","syslog\_pid":"31499","syslog\_message":"connect from unknown\[95.75.93.154\]\r","received\_at":"2016-01-05T12:22:55.674Z","received\_from":"0:0:0:0:0:0:0:1","syslog\_severity\_code":5,"syslog\_facility\_code":1,"syslog\_facility":"user-level","syslog\_severity":"notice"}

    }, {

      "\_index" : "logstash-2016.12.23",

      "\_type" : "syslog",

      "\_id" : "AVIRveM80svkz\_zfzuOQ",

      "\_score" : 0.06365098,

      "\_source":{"message":"Dec 23 14:42:56 louis named\[16000\]: client 199.48.164.7\#64817: query \(cache\) 'amsterdamboothuren.com/MX/IN' denied\r","@version":"1","@timestamp":"2016-12-23T06:42:56.000Z","host":"0:0:0:0:0:0:0:1","port":45093,"type":"syslog","syslog\_timestamp":"Dec 23 14:42:56","syslog\_hostname":"louis","syslog\_program":"named","syslog\_pid":"16000","syslog\_message":"client 199.48.164.7\#64817: query \(cache\) 'amsterdamboothuren.com/MX/IN' denied\r","received\_at":"2016-01-05T12:23:22.809Z","received\_from":"0:0:0:0:0:0:0:1","syslog\_severity\_code":5,"syslog\_facility\_code":1,"syslog\_facility":"user-level","syslog\_severity":"notice"}

    } \]

  }

}

\[root@h102 etc\]\# 

多行日志事件

类似于mysql slow log 这一类的日志并非一次一行，而是多行



Logstash 也可以处理，只是目前此功能还比较弱



配置如下



\[root@h102 etc\]\# cat logstash-multiline.conf

input {

  stdin {

    codec =&gt; multiline {

      pattern =&gt; "^\# User@Host:"

      negate =&gt; true

      what =&gt; previous

    }

  }

}



output {

  elasticsearch { hosts =&gt; \["localhost:9200"\] }

  stdout { codec =&gt; rubydebug }

}

\[root@h102 etc\]\# time /opt/logstash/bin/logstash -f logstash-multiline.conf -t 

Configuration OK



real	0m18.807s

user	0m30.841s

sys	0m2.290s

\[root@h102 etc\]\# 

pattern 为正则匹配

negate 为反转，只能为 true 或 false , 默认为 false ，代表不反转

what 为处理行为，只能为 previous 或 next ，为 previous 时，代表匹配此模式的行属于前面的事件内容，为 next 时，代表匹配此模式的行属于后面的事件内容

上面的配置表明，如果不以 \# User@Host: 开头的行都属于前面的事件内容



开启 Logstash 进行测试



\[root@h102 etc\]\# time /opt/logstash/bin/logstash -f logstash-multiline.conf 

Settings: Default filter workers: 1

Logstash startup completed

\# Time: 150710 16:37:53

\# User@Host: root\[root\] @ localhost \[\]

{

    "@timestamp" =&gt; "2016-01-05T14:01:57.953Z",

       "message" =&gt; "\# Time: 150710 16:37:53",

      "@version" =&gt; "1",

          "host" =&gt; "h102.temp"

}

\# Thread\_id: 113  Schema: mysqlslap  Last\_errno: 0  Killed: 0

\# Query\_time: 1.134132  Lock\_time: 0.000029  Rows\_sent: 1  Rows\_examined: 1  Rows\_affected: 0  Rows\_read: 1

\# Bytes\_sent: 2168

SET timestamp=1436517473;

SELECT intcol1,intcol2,intcol3,intcol4,intcol5,charcol1,charcol2,charcol3,charcol4,charcol5,charcol6,charcol7,charcol8,charcol9,charco

l10 FROM t1 WHERE id =  '31';

\# User@Host: root\[root\] @ localhost \[\]

{

    "@timestamp" =&gt; "2016-01-05T14:02:03.773Z",

       "message" =&gt; "\# User@Host: root\[root\] @ localhost \[\]\n\# Thread\_id: 113  Schema: mysqlslap  Last\_errno: 0  Killed: 0\n\# Query\_time: 1.134132  Lock\_time: 0.000029  Rows\_sent: 1  Rows\_examined: 1  Rows\_affected: 0  Rows\_read: 1\n\# Bytes\_sent: 2168\nSET timestamp=1436517473;\nSELECT intcol1,intcol2,intcol3,intcol4,intcol5,charcol1,charcol2,charcol3,charcol4,charcol5,charcol6,charcol7,charcol8,charcol9,charco\nl10 FROM t1 WHERE id =  '31';",

      "@version" =&gt; "1",

          "tags" =&gt; \[

        \[0\] "multiline"

    \],

          "host" =&gt; "h102.temp"

}

\# Thread\_id: 110  Schema: mysqlslap  Last\_errno: 0  Killed: 0

\# Query\_time: 1.385901  Lock\_time: 0.000037  Rows\_sent: 1  Rows\_examined: 1  Rows\_affected: 0  Rows\_read: 1

\# Bytes\_sent: 2167

SET timestamp=1436517473;

SELECT intcol1,intcol2,intcol3,intcol4,intcol5,charcol1,charcol2,charcol3,charcol4,charcol5,charcol6,charcol7,charcol8,charcol9,charco

l10 FROM t1 WHERE id =  '43';

\# User@Host: root\[root\] @ localhost \[\]

{

    "@timestamp" =&gt; "2016-01-05T14:02:51.114Z",

       "message" =&gt; "\# User@Host: root\[root\] @ localhost \[\]\n\# Thread\_id: 110  Schema: mysqlslap  Last\_errno: 0  Killed: 0\n\# Query\_time: 1.385901  Lock\_time: 0.000037  Rows\_sent: 1  Rows\_examined: 1  Rows\_affected: 0  Rows\_read: 1\n\# Bytes\_sent: 2167\nSET timestamp=1436517473;\nSELECT intcol1,intcol2,intcol3,intcol4,intcol5,charcol1,charcol2,charcol3,charcol4,charcol5,charcol6,charcol7,charcol8,charcol9,charco\nl10 FROM t1 WHERE id =  '43';",

      "@version" =&gt; "1",

          "tags" =&gt; \[

        \[0\] "multiline"

    \],

          "host" =&gt; "h102.temp"

}

发现在输入 \# User@Host: 之前，所有的行都被进行压栈处理，输入此条信息后，前面的信息进行了一个完结，又重新等待新的输入，直到遇到又一个 \# User@Host:



Tip: 暂时没有很好的办法处理诸如 \# Time: 150710 16:37:53 的行，这样的行被算在了前一条的事件日志中



命令汇总

java -version

/opt/logstash/bin/logstash -e 'input { stdin { } } output { stdout {} }'

cat first-pipeline.conf

/opt/logstash/bin/logstash -f first-pipeline.conf -t

/opt/logstash/bin/logstash -f first-pipeline.conf

curl -XGET 'localhost:9200/logstash-2015.12.23/\_search?q=response=404'

curl -XGET 'localhost:9200/logstash-2015.12.23/\_search?q=response=304&pretty'

curl -XGET 'localhost:9200/logstash-2015.12.23/\_search?q=geoip.city\_name=Buffalo&pretty'

grep -v "\#" /etc/filebeat/filebeat.yml \| grep -v "^$"

/etc/init.d/filebeat start

/etc/init.d/filebeat status

cat logstash-filebeat-es-simple.conf

/opt/logstash/bin/logstash -f logstash-filebeat-es-simple.conf

curl localhost:9200/\_cat/indices?v

curl -XGET 'localhost:9200/filebeat-2016.01.05/\_search?q=message=2935&pretty'

cat logstash-syslog.conf

/opt/logstash/bin/logstash -f logstash-syslog.conf

telnet localhost 5000

curl -XGET 'localhost:9200/logstash-2016.12.23/\_search?q=message=louis&pretty'

cat logstash-multiline.conf

time /opt/logstash/bin/logstash -f logstash-multiline.conf





