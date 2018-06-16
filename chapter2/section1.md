# 多行日志事件

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





