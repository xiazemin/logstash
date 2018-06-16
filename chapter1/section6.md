Logstash 基础

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



