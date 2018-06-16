# Filebeat

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

