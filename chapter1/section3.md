# logstash实现日志文件同步到elasticsearch

1、目的：

将本地磁盘存储的日志文件同步（全量同步、实时增量同步）到ES中。 

这里写图片描述



2、源文件：

\[root@5b9dbaaa148a test\_log\]\# ll

-rwxrwxrwx 1 root root 170 Jul 5 08:02 logmachine.sh

-rw-r--r-- 1 root root 66 Jul 5 08:25 MProbe01.log

-rw-r--r-- 1 root root 74 Jul 5 08:28 MProbe02.log

1

2

3

4

3、增量实时同步脚本：

\[root@5b9dbaaa148a test\_log\]\# cat logmachine.sh

\#!/bin/bash

icnt=0;

while \(true\)

do

  echo "\[debug\]\[20160703-15:00\]"$icnt &gt;&gt; MProbe01.log

  echo "\[ERROR\]\[20160704-17:00\]"$icnt &gt;&gt; MProbe02.log

  icnt=$\(\(icnt+1\)\);

done

1

2

3

4

5

6

7

8

9

4、logstash配置文件：

\[root@5b9dbaaa148a logstash\_jdbc\_test\]\# cat log\_test.conf

input {

  file {

  path=&gt; \[ "/usr/local/logstash/bin/test\_log/MProbe01.log",

"/usr/local/logstash/bin/test\_log/MProbe02.log" \]

  \#codec=&gt;multiline {

  \# pattern =&gt; "^\s"

  \# what=&gt;"previous"

  \#}

  type=&gt;"probe\_log"  \#类型名称

  \# tags=&gt;\["XX.XX.XX.XX"\]

  }

}



\#\#\#过滤

\#filter{

\# grok {

\# match =&gt; \["message","mailmonitor"\]

\# add\_tag =&gt; \[mailmonitor\]

\# }



\# grok {

\# match =&gt; \[ "message", "smsmonitor" \]

\# add\_tag =&gt; \[smsmonitor\]

\# }

\# ....

\#}



\#\#\#output to es

output {

  elasticsearch {

  hosts =&gt; "10.8.5.101:9200"

  index =&gt; "mprobe\_index"     \#索引名称

  \#template\_name =&gt; "mprobelog"

  \#document\_id =&gt; "%{id}"

  }

  stdout { codec =&gt; json\_lines }

}

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

5、同步测试：



\[root@5b9dbaaa148a bin\]\# ./logstash -f ./logstash\_jdbc\_test/log\_test.conf

Settings: Default pipeline workers: 24

Pipeline main started

{"message":"\[DEbug\]\[20160305-15:35\]testing02","@version":"1","@timestamp":"2016-07-05T07:26:08.043Z","path":"/usr/local/logstash/bin/test\_log/MProbe01.log","host":"5b9dbaaa148a"

1

2

3

4

6、结果验证 

（1）日志记录：



\[root@5b9dbaaa148a test\_log\]\# tail -f MProbe01.log

\[DEbug\]\[20160305-15:35\]testing02

\[DEbug\]\[20160305-15:35\]testing01

^C

\[root@5b9dbaaa148a test\_log\]\# tail -f MProbe02.log

\[DEbug\]\[20160305-15:35\]testing02\_001

\[DEbug\]\[20160305-15:35\]testing02\_003



（2）ES记录 

这里写图片描述





