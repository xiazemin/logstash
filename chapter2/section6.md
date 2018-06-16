# logstash配置

\[root@h102 etc\]\# cat logstash-for-mongo.conf  

input {

  stdin {}

  file {

	    type=&gt;"mongolog"

	    path=&gt;"/tmp/xyz.log"

	    start\_position =&gt; beginning

       }

}



filter {

  grok {

       match =&gt; \["message","%{TIMESTAMP\_ISO8601:timestamp}\s+%{MONGO3\_SEVERITY:severity}\s+%{MONGO3\_COMPONENT:component}%{SPACE}\(?:\\[%{DATA:context}\\]\)?\s+%{GREEDYDATA:body}"\]

  } 

  if \[body\] =~ "ms$"  {  

       grok {

	match =&gt; \["body",".\*\}\(\s+%{NUMBER:spend\_time:int}ms$\)?"\]

       }

 }

 date {

   match =&gt; \[ "timestamp", "ISO8601" \]

   \#remove\_field =&gt; \[ "timestamp" \]

  }

}



output {

  elasticsearch { 

  	hosts =&gt; \["localhost:9200"\] 

        index=&gt;"mongodb-slow-log-%{+YYYY.MM.dd}"

	}

  stdout { codec =&gt; rubydebug }

}

\[root@h102 etc\]\# 

检测配置

\[root@h102 etc\]\# /opt/logstash/bin/logstash -f  logstash-for-mongo.conf -t

Configuration OK

\[root@h102 etc\]\# 

运行logstash

\[root@h102 etc\]\# /opt/logstash/bin/logstash -f  logstash-for-mongo.conf

Settings: Default filter workers: 1

Logstash startup completed

...

...

...

输入测试

2015-12-25T18:41:47.683+0800 I CONTROL  \[signalProcessingThread\] db version v3.0.3

{

       "message" =&gt; "2015-12-25T18:41:47.683+0800 I CONTROL  \[signalProcessingThread\] db version v3.0.3",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-25T10:41:47.683Z",

          "host" =&gt; "h102.temp",

     "timestamp" =&gt; "2015-12-25T18:41:47.683+0800",

      "severity" =&gt; "I",

     "component" =&gt; "CONTROL",

       "context" =&gt; "signalProcessingThread",

          "body" =&gt; "db version v3.0.3"

}

2015-12-25T20:54:11.336+0800 I JOURNAL  \[journal writer\] old journal file will be removed: /var/lib/mongo/journal/j.\_177

{

       "message" =&gt; "2015-12-25T20:54:11.336+0800 I JOURNAL  \[journal writer\] old journal file will be removed: /var/lib/mongo/journal/j.\_177",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-25T12:54:11.336Z",

          "host" =&gt; "h102.temp",

     "timestamp" =&gt; "2015-12-25T20:54:11.336+0800",

      "severity" =&gt; "I",

     "component" =&gt; "JOURNAL",

       "context" =&gt; "journal writer",

          "body" =&gt; "old journal file will be removed: /var/lib/mongo/journal/j.\_177"

}

2015-12-26T00:46:36.512+0800 I COMMAND  \[conn424487\] command feed\_test\_repo.$cmd command: geoNear { geoNear: "users", near: \[ 88.598884, 44.102866 \], query: {}, num: 30, maxDistance: 10 } keyUpdates:0 writeConflicts:0 numYields:399 reslen:37700 locks:{ Global: { acquireCount: { r: 400 } }, MMAPV1Journal: { acquireCount: { r: 400 } }, Database: { acquireCount: { r: 400 } }, Collection: { acquireCount: { R: 400 } } } 2584ms

{

       "message" =&gt; "2015-12-26T00:46:36.512+0800 I COMMAND  \[conn424487\] command feed\_test\_repo.$cmd command: geoNear { geoNear: \"users\", near: \[ 88.598884, 44.102866 \], query: {}, num: 30, maxDistance: 10 } keyUpdates:0 writeConflicts:0 numYields:399 reslen:37700 locks:{ Global: { acquireCount: { r: 400 } }, MMAPV1Journal: { acquireCount: { r: 400 } }, Database: { acquireCount: { r: 400 } }, Collection: { acquireCount: { R: 400 } } } 2584ms",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-25T16:46:36.512Z",

          "host" =&gt; "h102.temp",

     "timestamp" =&gt; "2015-12-26T00:46:36.512+0800",

      "severity" =&gt; "I",

     "component" =&gt; "COMMAND",

       "context" =&gt; "conn424487",

          "body" =&gt; "command feed\_test\_repo.$cmd command: geoNear { geoNear: \"users\", near: \[ 88.598884, 44.102866 \], query: {}, num: 30, maxDistance: 10 } keyUpdates:0 writeConflicts:0 numYields:399 reslen:37700 locks:{ Global: { acquireCount: { r: 400 } }, MMAPV1Journal: { acquireCount: { r: 400 } }, Database: { acquireCount: { r: 400 } }, Collection: { acquireCount: { R: 400 } } } 2584ms",

    "spend\_time" =&gt; 2584

}

可以正常解析



Tip: 如果无法正常解析, tags 里会多出一个 \_grokparsefailure ，并且无法捕获下面多出来的那些值



18:41:47.683+0800 I CONTROL  \[signalProcessingThread\] db version v3.0.3

{

       "message" =&gt; "18:41:47.683+0800 I CONTROL  \[signalProcessingThread\] db version v3.0.3",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2016-02-15T07:20:19.479Z",

          "host" =&gt; "h102.temp",

          "tags" =&gt; \[

        \[0\] "\_grokparsefailure"

    \]

}

配置分析

input

input {

  stdin {}

  file {

	    type=&gt;"mongolog"

	    path=&gt;"/tmp/xyz.log"

	    start\_position =&gt; beginning

       }

}

Item	Comment

input {	框定输入源的定义范围

stdin {	定义了一个输入源，使用 stdin 插件从标准输入读取数据，也就是终端读入\(生产中不会这样配置，一般用来进行交互调试\)

file {	定义了一个输入源，使用 file 插件从指定文本读取数据

type=&gt;"mongolog"	指定读入数据的类型

path=&gt;"/tmp/xyz.log"	指定输入源文件的地址，必须为绝对路径

start\_position =&gt; beginning	指定读取特性，默认为跟踪新生成的记录或条目

合起来的意思就是：从终端读取，从 /tmp/xyz.log 的开头读取并打上mongolog的类型\(从终端读取的没有此类型标签\)



filter

filter {

  grok {

       match =&gt; \["message","%{TIMESTAMP\_ISO8601:timestamp}\s+%{MONGO3\_SEVERITY:severity}\s+%{MONGO3\_COMPONENT:component}%{SPACE}\(?:\\[%{DATA:context}\\]\)?\s+%{GREEDYDATA:body}"\]

  } 

  if \[body\] =~ "ms$"  {  

       grok {

	     match =&gt; \["body",".\*\}\(\s+%{NUMBER:spend\_time:int}ms$\)?"\]

       }

 }

 date {

   match =&gt; \[ "timestamp", "ISO8601" \]

   \#remove\_field =&gt; \[ "timestamp" \]

  }

}

Item	Comment

filter {	框定处理逻辑的定义范围

grok {	定义了一个过滤器，使用 grok 插件来解析文本，和抓取信息，用于文本结构化

match =&gt; \["message",".\*"\]	用来match哈希 {"message" =&gt; ".\*patten.\*"},然后把正则捕获的值作为事件日志的filed

if \[body\] =~ "ms$"	判断 body 字段中是否以 ms 结尾，如果匹配，就执行定义的代码段

match =&gt; \["body",".\*\}\(\s+%{NUMBER:spend\_time:int}ms$\)?"\]	尝试从body中抽取花费的时间

date {	定义了一个过滤器，使用 date 插件来从fileds中解析出时间，然后把获取的时间值作为此次事件日志的时间戳

match =&gt; \[ "timestamp", "ISO8601" \]	取用 timestamp 中的时间作为事件日志时间戳，模式匹配为 ISO8601

\#remove\_field =&gt; \[ "timestamp" \]	一般而言，日志会有一个自己的时间戳 @timestamp ,这是logstash或 beats看到日志时的时间点，但是上一步已经将从日志捕获的时间赋给了 @timestamp ，所以 timestamp 就是一份冗余的信息,可以使用 remove\_field 方法来删掉这个字段，但我选择保留

Note: 这里的 if 判断不能省，否则会产生大量的 \_grokparsefailure



output

output {

  elasticsearch { 

  	hosts =&gt; \["localhost:9200"\] 

        index=&gt;"mongodb-slow-log-%{+YYYY.MM.dd}"

	}

  stdout { codec =&gt; rubydebug }

}

Item	Comment

output {	框定出口的定义范围

elasticsearch {	定义了一个出口，使用 elasticsearch 插件来进行输出，将结果输出到ES中

hosts =&gt; \["localhost:9200"\]	指定es的目标地址为 localhost:9200

index=&gt;"mongodb-slow-log-%{+YYYY.MM.dd}"	指定存到哪个index，如不指定，默认为logstash-%{+YYYY.MM.dd}

stdout { codec =&gt; rubydebug }	定义了一个出口，使用 stdout 插件将信息输出到标准输，也就是终端，并且使用 rubydebug 插件处理过后进行展示，也就是行成jason格式 \(生产不会这样配置，一般用来进行交互调试\)

正则

%{TIMESTAMP\_ISO8601:timestamp}\s+%{MONGO3\_SEVERITY:severity}\s+%{MONGO3\_COMPONENT:component}%{SPACE}\(?:\\[%{DATA:context}\\]\)?\s+%{GREEDYDATA:body}



Item	Comment

%{TIMESTAMP\_ISO8601:timestamp}	将匹配 TIMESTAMP\_ISO8601 模式的结果放到 timestamp 中

\s+	匹配一个或多个空字符

\(?:\\[%{DATA:context}\\]\)?	内容可能有，也可能无，如果有,以 \[ 开头，且以 \] 结尾，中间的任何内容放到 context 中

Tip: 可以参考 mongodb patterns 中的匹配设置 ，MONGO3\_LOG %{TIMESTAMP\_ISO8601:timestamp} %{MONGO3\_SEVERITY:severity} %{MONGO3\_COMPONENT:component}%{SPACE}\(?:\\[%{DATA:context}\\]\)? %{GREEDYDATA:message} ,我将最后的部分存入了body，不然会存到原来的 message 字段中， 使message变成一个列表，内容变成 message中的第二个元素，然后将空格替换成了 \s+ ，这样会更消耗计算资源，但是更严谨



.\*\}\(\s+%{NUMBER:spend\_time:int}ms$\)?



Item	Comment

.\*	匹配任意内容

\}	匹配 }

\(\s+%{NUMBER:spend\_time:int}ms$\)?	内容可能有，也可能无，如果有,以若干个空字符开头，以 ms 结尾，将中间的 int 类型数值存储到 spend\_time 中

Note: \} 不能省，否则以 .\* 的贪婪特性会一口气将后面的所有内容都吞噬掉，从而使 %{NUMBER:spend\_time:int} 匹配不到数据





