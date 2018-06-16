# 安装logstash

\[root@h102 ELK\]\# rpm -ivh logstash-2.1.1-1.noarch.rpm 

Preparing...                \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# \[100%\]

   1:logstash               \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# \[100%\]

\[root@h102 ELK\]\# echo $?

0

\[root@h102 ELK\]\# 

默认情况下 logstash 被安装到了 /opt/logstash/ 下



\[root@h102 ~\]\# ll /opt/logstash/

total 148

drwxr-xr-x 2 logstash logstash  4096 Dec 22 20:25 bin

-rw-rw-r-- 1 logstash logstash 97345 Dec  8 02:04 CHANGELOG.md

-rw-rw-r-- 1 logstash logstash  2249 Dec  8 02:04 CONTRIBUTORS

-rw-rw-r-- 1 logstash logstash  3771 Dec  8 02:04 Gemfile

-rw-rw-r-- 1 logstash logstash 21837 Dec  8 02:04 Gemfile.jruby-1.9.lock

drwxr-xr-x 4 logstash logstash  4096 Dec 22 20:25 lib

-rw-rw-r-- 1 logstash logstash   589 Dec  8 02:04 LICENSE

-rw-rw-r-- 1 logstash logstash   149 Dec  8 02:04 NOTICE.TXT

drwxr-xr-x 4 logstash logstash  4096 Dec 22 20:25 vendor

\[root@h102 ~\]\# tree /opt/logstash/bin/

/opt/logstash/bin/

├── logstash

├── logstash.bat

├── logstash.lib.sh

├── plugin

├── plugin.bat

├── rspec

├── rspec.bat

└── setup.bat



0 directories, 8 files

\[root@h102 ~\]\# 

logstash的简单测试

\[root@h102 ELK\]\# /opt/logstash/bin/logstash -e 'input { stdin { } } output { stdout {} }'

Settings: Default filter workers: 1

Logstash startup completed

simple test

2015-12-22T12:30:32.996Z h102.temp simple test

getting started with logstash

2015-12-22T12:30:48.264Z h102.temp getting started with logstash

hello example

2015-12-22T12:31:08.384Z h102.temp hello example

...

...

输完以上命令后，要稍等一下， Logstash startup completed 字样出现后，才表示 logstash 已经启动，之后的输入，才会获得相应响应



Tip: The -e flag enables you to specify a configuration directly from the command line. Specifying configurations at the command line lets you quickly test configurations without having to edit a file between iterations. This pipeline takes input from the standard input, stdin , and moves that input to the standard output, stdout, in a structured format .



此时系统中多了一个给 logstash 用的java进程



\[root@h102 ~\]\# ps  faux \| grep java

root     34241 26.4  8.5 2506504 164332 pts/0  Sl+  20:28   0:53  \|       \\_ /usr/bin/java -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -Djava.awt.headless=true -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+HeapDumpOnOutOfMemoryError -Xmx1g -Xss2048k -Djffi.boot.library.path=/opt/logstash/vendor/jruby/lib/jni -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -Djava.awt.headless=true -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/opt/logstash/heapdump.hprof -Xbootclasspath/a:/opt/logstash/vendor/jruby/lib/jruby.jar -classpath : -Djruby.home=/opt/logstash/vendor/jruby -Djruby.lib=/opt/logstash/vendor/jruby/lib -Djruby.script=jruby -Djruby.shell=/bin/sh org.jruby.Main --1.9 /opt/logstash/lib/bootstrap/environment.rb logstash/runner.rb agent -e input { stdin { } } output { stdout {} }

root     34356  0.0  0.0 103256   828 pts/2    S+   20:32   0:00          \\_ grep java

488       8738  2.3 14.8 2527948 284032 ?      Sl   19:31   1:26 /usr/bin/java -Xms256m -Xmx1g -Djava.awt.headless=true -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+HeapDumpOnOutOfMemoryError -XX:+DisableExplicitGC -Dfile.encoding=UTF-8 -Djna.nosys=true -Des.path.home=/usr/share/elasticsearch -cp /usr/share/elasticsearch/lib/elasticsearch-2.1.1.jar:/usr/share/elasticsearch/lib/\* org.elasticsearch.bootstrap.Elasticsearch start -p /var/run/elasticsearch/elasticsearch.pid -d -Des.default.path.home=/usr/share/elasticsearch -Des.default.path.logs=/var/log/elasticsearch -Des.default.path.data=/var/lib/elasticsearch -Des.default.path.conf=/etc/elasticsearch

\[root@h102 ~\]\#

logstash的配置文件

\[root@h102 etc\]\# vim logstash-simple.conf

\[root@h102 etc\]\# cat logstash-simple.conf 

input {stdin{}}

output {

	stdout {codec =&gt; rubydebug}

} 

\[root@h102 etc\]\# /opt/logstash/bin/logstash -f logstash-simple.conf --configtest

Configuration OK

\[root@h102 etc\]\# time /opt/logstash/bin/logstash -f logstash-simple.conf --configtest

Configuration OK



real	0m18.992s

user	0m32.038s

sys	0m1.425s

\[root@h102 etc\]\# 

使用 --configtest 参数可以对配置文件进行检查，但感觉这么点内容，要花这么长时间，有点不可思议



\(相较而言Nginx的检查就在一瞬间，是由于JVM的启停么，专门为检查配置写个脚本也要不了多少代码吧\)



检查好后，可以使用 -f 直接运行了



\[root@h102 etc\]\# /opt/logstash/bin/logstash -f logstash-simple.conf

Settings: Default filter workers: 1

Logstash startup completed

simple test

{

       "message" =&gt; "simple test",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-22T13:10:11.729Z",

          "host" =&gt; "h102.temp"

}

getting started with logstash

{

       "message" =&gt; "getting started with logstash",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-22T13:10:34.294Z",

          "host" =&gt; "h102.temp"

}

hello example

{

       "message" =&gt; "hello example",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-22T13:10:39.486Z",

          "host" =&gt; "h102.temp"

}

...

...

效果还是很明显的，有了格式化的输出



basic\_logstash\_pipeline.png



Tip: A Logstash pipeline has two required elements, input and output , and one optional element, filter . The input plugins consume data from a source, the filter plugins modify the data as you specify, and the output plugins write the data to a destination.



退出运行

使用 Ctrl+C 退出 logstash



...

...

^CSIGINT received. Shutting down the pipeline. {:level=&gt;:warn}



Logstash shutdown completed

\[root@h102 ELK\]\# 

使用 Ctrl+D 退出 logstash



...

...

Logstash shutdown completed

\[root@h102 etc\]\# 

数据存到Elasticsearch

\[root@h102 etc\]\# vim  logstash-es-simple.conf

\[root@h102 etc\]\# cat logstash-es-simple.conf 

input {stdin{}}

output {

	elasticsearch {hosts=&gt;"localhost:9200"}

	stdout {codec=&gt;rubydebug}

}

\[root@h102 etc\]\# 

\[root@h102 etc\]\# /opt/logstash/bin/logstash -f logstash-es-simple.conf 

Settings: Default filter workers: 1

Logstash startup completed

simple test

{

       "message" =&gt; "simple test",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-22T13:35:33.252Z",

          "host" =&gt; "h102.temp"

}

getting started with logstash

{

       "message" =&gt; "getting started with logstash",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-22T13:35:47.999Z",

          "host" =&gt; "h102.temp"

}

hello example

{

       "message" =&gt; "hello example",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-22T13:35:58.383Z",

          "host" =&gt; "h102.temp"

}

...

...

貌似结果和前一种情况差不多，但此时ES里已经有数据了



\[root@h102 ~\]\# curl 'http://localhost:9200/\_search?pretty'

{

  "took" : 35,

  "timed\_out" : false,

  "\_shards" : {

    "total" : 6,

    "successful" : 6,

    "failed" : 0

  },

  "hits" : {

    "total" : 4,

    "max\_score" : 1.0,

    "hits" : \[ {

      "\_index" : ".kibana",

      "\_type" : "config",

      "\_id" : "4.3.1",

      "\_score" : 1.0,

      "\_source":{"buildNum":9517}

    }, {

      "\_index" : "logstash-2015.12.22",

      "\_type" : "logs",

      "\_id" : "AVHJ5yfWrEoC64-rdPBW",

      "\_score" : 1.0,

      "\_source":{"message":"getting started with logstash","@version":"1","@timestamp":"2015-12-22T13:35:47.999Z","host":"h102.temp"}

    }, {

      "\_index" : "logstash-2015.12.22",

      "\_type" : "logs",

      "\_id" : "AVHJ51ORrEoC64-rdPBX",

      "\_score" : 1.0,

      "\_source":{"message":"hello example","@version":"1","@timestamp":"2015-12-22T13:35:58.383Z","host":"h102.temp"}

    }, {

      "\_index" : "logstash-2015.12.22",

      "\_type" : "logs",

      "\_id" : "AVHJ5vLKrEoC64-rdPBV",

      "\_score" : 1.0,

      "\_source":{"message":"simple test","@version":"1","@timestamp":"2015-12-22T13:35:33.252Z","host":"h102.temp"}

    } \]

  }

}

\[root@h102 ~\]\# 

由于ES里已经有了数据，再看kibana，\[Create\] 变得可以点击了



kibana2.png



\[Create\] 点击完后，点击 \[Discover\] 就会看到有日志信息展示出来了



kibana3.png



继续在终端里输入测试信息，选择时间刷新后就会看到kibana的显示也发生了变化



kibana4.png



output 里多出来两条配置，其实代表可以同时指定多个输出，将结果写一份到ES，也写一份到终端

elasticsearch {hosts=&gt;"localhost:9200"}

stdout {codec=&gt;rubydebug}

使用 hosts 来指定ES的位置，老版使用的是 host ，如果在这里使用 host 会报错

可以使用 hosts =&gt; \[“IP Address 1:port1”, “IP Address 2:port2”, “IP Address 3”\] 的方式指定多个进行冗余，和负载均衡

如果ES使用的 9200 端口，是可以在配置里省略的



