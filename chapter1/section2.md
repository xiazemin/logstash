# logstash语法

logstash语法

http://www.ttlsa.com/elk/elk-logstash-configuration-syntax/

https://www.elastic.co/guide/en/logstash/current/event-dependent-configuration.html



logstash grok原理

参考:

https://www.kancloud.cn/hanxt/elk/155901

https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html



正则表达式参考:

https://github.com/kkos/oniguruma/blob/master/doc/RE



grok的意思: \(用感觉感知,而非动脑思考\)to understand sth completely using your feelings rather than considering the facts



这个目录下有各种定义好的正则字段

/usr/local/logstash/vendor/bundle/jruby/1.9/gems/logstash-patterns-core-4.1.2/patterns



或者直接访问这个:

https://github.com/elastic/logstash/blob/v1.4.2/patterns/grok-patterns





$  ls /usr/local/logstash/vendor/bundle/jruby/1.9/gems/logstash-patterns-core-4.1.2/patterns/

aws     bind  exim       grok-patterns  httpd  junos         maven        mcollective-patterns  nagios      rails  ruby

bacula  bro   firewalls  haproxy        java   linux-syslog  mcollective  mongodb               postgresql  redis  squid

如apache日志解析: logstash过滤解析apache日志



filter {

    grok {

        match =&gt; { "message" =&gt; "%{COMBINEDAPACHELOG}"}

    }

}

logstash内置的pattern的定义\(嵌套调用\)





再举个例子

%{IP:client} 这里意思是: 用IP正则去匹配日志内容,匹配到的内容存储在key client里.



input {

  file {

    path =&gt; "/var/log/http.log"

  }

}

filter {

  grok {

    match =&gt; { "message" =&gt; "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" }

  }

}

output {

    stdout { codec =&gt; rubydebug }

}

grok的remove\_field

参考:

https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html

https://doc.yonyoucloud.com/doc/logstash-best-practice-cn/filter/grok.html



我们只需要request\_time字段,默认仅match会读取message某字段赋给新字段,这样就造成了数据重复,为了解决这个问题,干掉message字段



input {stdin{}}

filter {

    grok {

        match =&gt; {

            "message" =&gt; "\s+\(?&lt;request\_time&gt;\d+\(?:\.\d+\)?\)\s+"

        }

    }

}

output {stdout{ codec =&gt; rubydebug }}



begin 123.456 end

{

        "@version" =&gt; "1",

            "host" =&gt; "ip-70.32.1.32.hosted.by.gigenet.com",

      "@timestamp" =&gt; 2017-11-29T03:47:15.377Z,

    "request\_time" =&gt; "123.456",

         "message" =&gt; "begin 123.456 end"

}

input {stdin{}}

filter {

    grok {

        match =&gt; {

            "message" =&gt; "\s+\(?&lt;request\_time&gt;\d+\(?:\.\d+\)?\)\s+"

        }

        remove\_field =&gt; \["message"\]

    }

}

output {stdout{ codec =&gt; rubydebug }}









begin 123.456 end

{

        "@version" =&gt; "1",

            "host" =&gt; "ip-70.32.1.32.hosted.by.gigenet.com",

      "@timestamp" =&gt; 2017-11-29T03:51:01.135Z,

    "request\_time" =&gt; "123.456"

}

自定义pattern

参考: https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html

可以写文件里,也可以直接指定,如上一个例子.



$ cat /var/sample.log

Jan  1 06:25:43 mailserver14 postfix/cleanup\[21403\]: BEF25A72965: message-id=&lt;20130101142543.5828399CCAF@mailserver14.example.com&gt;



$ cat ./patterns/postfix:

POSTFIX\_QUEUEID \[0-9A-F\]{10,11}



input {

  file {

    path =&gt; "/var/sample.log"

  }

}

filter {

  grok {

    patterns\_dir =&gt; \["./patterns"\]

    match =&gt; { "message" =&gt; "%{SYSLOGBASE} %{POSTFIX\_QUEUEID:queue\_id}: %{GREEDYDATA:syslog\_message}" }

  }

}

output {

    stdout { codec =&gt; rubydebug }

}

grok解析apache日志,并修改date格式

参考:http://blog.51cto.com/irow10/1828077 这里格式有问题,我修复了.



input {

  stdin {}

}

filter {

    grok {

      match =&gt; { "message" =&gt; "%{IPORHOST:addre} %{USER:ident} %{USER:auth} \\[%{HTTPDATE:timestamp}\\] \"%{WORD:http\_method} %{NOTSPACE:request} HTTP/%{NUMBER:httpversion}\" %{NUMBER:status} \(?:%{NUMBER:bytes}\|-\) \"\(?:%{URI:http\_referer}\|-\)\" \"%{GREEDYDATA:User\_Agent}\"" }

      remove\_field =&gt; \["message"\]

    }

    date {

      match =&gt; \[ "timestamp", "dd/MMM/YYYY:HH:mm:ss Z" \]

    }

}



output {

    stdout { codec =&gt; rubydebug }

}

192.168.10.97 - - \[19/Jul/2016:16:28:52 +0800\] "GET / HTTP/1.1" 200 23 "-" "Mozilla/5.0 \(Windows NT 6.1; WOW64\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/45.0.2454.101 Safari/537.36"

{

        "request" =&gt; "/",

           "auth" =&gt; "-",

          "ident" =&gt; "-",

     "User\_Agent" =&gt; "Mozilla/5.0 \(Windows NT 6.1; WOW64\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/45.0.2454.101 Safari/537.36",

          "addre" =&gt; "192.168.10.97",

     "@timestamp" =&gt; 2016-07-19T08:28:52.000Z,

    "http\_method" =&gt; "GET",

          "bytes" =&gt; "23",

       "@version" =&gt; "1",

           "host" =&gt; "no190.pp100.net",

    "httpversion" =&gt; "1.1",

      "timestamp" =&gt; "19/Jul/2016:16:28:52 +0800",

         "status" =&gt; "200"

}

grok在线检测

参考: http://grokdebug.herokuapp.com/



192.168.10.97 - - \[19/Jul/2016:16:28:52 +0800\] "GET / HTTP/1.1" 200 23 "-" "Mozilla/5.0 \(Windows NT 6.1; WOW64\) AppleWebKit/537.36 \(KHTML, like Gecko\) Chrome/45.0.2454.101 Safari/537.36"



%{IPORHOST:addre} %{USER:ident} %{USER:auth} \\[%{HTTPDATE:timestamp}\\] \"%{WORD:http\_method} %{NOTSPACE:request} HTTP/%{NUMBER:httpversion}\" %{NUMBER:status} \(?:%{NUMBER:bytes}\|-\) \"\(?:%{URI:http\_referer}\|-\)\" \"%{GREEDYDATA:User\_Agent}\"

logstash mutate插件-给整个条目添加个字段

参考: https://www.elastic.co/guide/en/logstash/current/event-dependent-configuration.html



input { stdin { } }



filter {

  mutate { add\_field =&gt; { "show" =&gt; "This data will be in the output" } }

}



output {

  if \[@metadata\]\[test\] == "Hello" {

    stdout { codec =&gt; rubydebug }

  }

}

sdf

{

      "@version" =&gt; "1",

          "host" =&gt; "ip-70.32.1.32.hosted.by.gigenet.com",

          "show" =&gt; "This data will be in the output",

    "@timestamp" =&gt; 2017-11-29T09:23:44.160Z,

       "message" =&gt; "sdf"

}

logstash input添加字段-add\_field

参考: http://www.21yunwei.com/archives/5296



input {

  file {

        path =&gt; "/logs/nginx/access.log"

        type =&gt; "nginx"

        start\_position =&gt; "beginning"

        add\_field =&gt; { "key"=&gt;"value"}

        codec =&gt; "json"

}

 

}

output {

stdout{

        codec =&gt; rubydebug{ }

}

}

logstash 5大插件--待了解

参考:

http://blog.51cto.com/irow10/1828077

https://segmentfault.com/a/1190000011721483

https://www.elastic.co/guide/en/logstash/current/plugins-filters-kv.html



date插件可以对日期格式定义

mutate插件可以增删字段,可以改写字段格式

kv插件可...



使用上面的日志作为示例，使用 mutate 插件的 lowercase 配置选项，我们可以将“log-level”字段转换为小写：



filter { 

    grok {...}

    mutate {   lowercase =&gt; \[ "log-level" \]  }

}

kv filter 来指示 Logstash 如何处理它

kv插件可以拆解



filter {  

kv {

source =&gt; "metadata"

trim =&gt; "\""

include\_keys =&gt; \[ "level","service","customerid",”queryid” \]

target =&gt; "kv"

    }

}

