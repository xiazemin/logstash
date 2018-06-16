# 流水线模型

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



