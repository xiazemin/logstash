# 从文件获取数据

生产环境中不太可能手动生成日志\(使用人肉输入到stdin的方式\)，而更多是从一个源日志文件那里读取



\[root@h102 etc\]\# vim logstash-file-es-simple.conf

\[root@h102 etc\]\# cat logstash-file-es-simple.conf

input {

	stdin{}

	file {

	    type=&gt;"syslog"

	    path=&gt;"/var/log/messages"

	    start\_position =&gt; beginning

	}

}

output {

	elasticsearch {hosts=&gt;"localhost:9200"}

	stdout {codec=&gt;rubydebug}

}

\[root@h102 etc\]\# /opt/logstash/bin/logstash -f logstash-file-es-simple.conf  -t

Configuration OK

\[root@h102 etc\]\# /opt/logstash/bin/logstash -f logstash-file-es-simple.conf  

Settings: Default filter workers: 1

Logstash startup completed

{

       "message" =&gt; "Dec 22 17:34:02 h102 rsyslogd: \[origin software=\"rsyslogd\" swVersion=\"5.8.10\" x-pid=\"1693\" x-info=\"http://www.rsyslog.com\"\] rsyslogd was HUPed",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-22T14:07:12.146Z",

          "host" =&gt; "h102.temp",

          "path" =&gt; "/var/log/messages",

          "type" =&gt; "syslog"

}

{

       "message" =&gt; "Dec 22 18:19:23 h102 dhclient\[1624\]: DHCPREQUEST on eth3 to 192.168.1.2 port 67 \(xid=0x58532bb2\)",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-22T14:07:12.148Z",

          "host" =&gt; "h102.temp",

          "path" =&gt; "/var/log/messages",

          "type" =&gt; "syslog"

}

{

       "message" =&gt; "Dec 22 18:19:23 h102 dhclient\[1624\]: DHCPACK from 192.168.1.2 \(xid=0x58532bb2\)",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-22T14:07:12.149Z",

          "host" =&gt; "h102.temp",

          "path" =&gt; "/var/log/messages",

          "type" =&gt; "syslog"

}

{

       "message" =&gt; "Dec 22 18:19:24 h102 dhclient\[1624\]: bound to 192.168.1.117 -- renewal in 3538 seconds.",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-22T14:07:12.150Z",

          "host" =&gt; "h102.temp",

          "path" =&gt; "/var/log/messages",

          "type" =&gt; "syslog"

}

{

       "message" =&gt; "Dec 22 18:27:04 h102 kernel: hrtimer: interrupt took 6428893 ns",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-22T14:07:12.150Z",

          "host" =&gt; "h102.temp",

          "path" =&gt; "/var/log/messages",

          "type" =&gt; "syslog"

}

{

       "message" =&gt; "Dec 22 19:18:22 h102 dhclient\[1624\]: DHCPREQUEST on eth3 to 192.168.1.2 port 67 \(xid=0x58532bb2\)",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-22T14:07:12.151Z",

          "host" =&gt; "h102.temp",

          "path" =&gt; "/var/log/messages",

          "type" =&gt; "syslog"

}

...

...

{

       "message" =&gt; "Dec 22 21:51:56 h102 dhclient\[1624\]: DHCPACK from 192.168.1.2 \(xid=0x58532bb2\)",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-22T14:07:12.188Z",

          "host" =&gt; "h102.temp",

          "path" =&gt; "/var/log/messages",

          "type" =&gt; "syslog"

}

{

       "message" =&gt; "Dec 22 21:51:57 h102 dhclient\[1624\]: bound to 192.168.1.117 -- renewal in 2973 seconds.",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-22T14:07:12.203Z",

          "host" =&gt; "h102.temp",

          "path" =&gt; "/var/log/messages",

          "type" =&gt; "syslog"

}

abc

{

       "message" =&gt; "abc",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-22T14:11:19.994Z",

          "host" =&gt; "h102.temp"

}

xyz

{

       "message" =&gt; "xyz",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-22T14:11:22.893Z",

          "host" =&gt; "h102.temp"

}

def

{

       "message" =&gt; "def",

      "@version" =&gt; "1",

    "@timestamp" =&gt; "2015-12-22T14:11:25.633Z",

          "host" =&gt; "h102.temp"

}

...

...

发现kibana里同时看到了所有的系统日志和手工测试数据



kibana5.png



start\_position =&gt; beginning 的作用是从头开始读数据，如果不加这个配置，就会产生类似 tail -f /var/log/messages 的效果，只对新生成的数据进行跟踪，此刻以前的都直接忽略，此配置得在具体环境下考虑使用与否

致此，ELK基本的搭建与操作就完成了

