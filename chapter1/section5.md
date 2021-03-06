# logstash之file input插件实现分析

首先看下一般的log框架是如何输出日志的：



可能是这样的：a.log.1,  a.log.2, a.log.3, a.log.4, a.log.5 循环输出；



可能是这样的： a.2014-5-5.log, a.2014-5-6.log, a.2014-5-7.log，每天生成一个日志文件；



可能是这样的：log.out，每次重启都会生成一个新的log.out，覆盖旧的文件。







那么，我们有哪些方面要实现和注意的？



提供正则或者globs方式的通配符。

要能判断文件是不是新建的。

如何判断文件有没有更新？

如何保存文件的读取进度？

如果我们在读取文件的过程中，文件被删除了会怎样？

如果我们在读取文件过程中，进程挂了，读取进度有没有及时保存？

在保存文件进度时，如果挂了，重启能不能正确恢复文件进度？

能不能保证读取的内容不重复？

如果日志文件很快生成，又很快删除了，是否能保证不漏掉？

如果日志文件是软链接\(soft link\)，能不能正确处理？

文件系统的inode会被回收利用，能不能处理这个？

有没有控制读进内存的数据的大小，防止占用过多的内存？

logstash的实现

下面解释下logstash是如何实现和处理上面的问题的：



可以配置path参数（Array），其中支持globs风格的匹配，如：



\[ruby\] view plain copy

path =&gt; \[ "/var/log/messages", "/var/log/\*.log" \]  

可以配置exclude参数（Array），排除掉不需要的文件，如：



\[ruby\] view plain copy

exclude =&gt; "\*.gz"  

利用inode来识别新文件

logstash把进度保存到所谓的sincedb里，实际上即这样的一个文本文件，默认是放在home目录下的，如：



.sincedb\_e794081d6134aace51b759aea8cc3be2



.sincedb\_f7a0c8a0def03e0c572511ceea0b9f63



后面是日志文件，即path的hash值。这样就区分了不同的文件名的日志文件的进度保存问题。



sincedb文件里是类似这样的内容：



6511055 0 2051 118617881

5495516 0 2051 155859036

6348913 0 2051 148511449



上面的4列分别是：



inode, major number, minor number, pos。



其中major number和minor number是设备相关的数字，参考：http://unix.stackexchange.com/questions/73988/linux-major-and-minor-device-numbers



inode是文件系统给文件分配的是一个号码，参考：http://zh.wikipedia.org/wiki/Inode



因此logstash区分了设备，文件名，文件的不同版本。



这里引出了一个新问题，用inode来判断文件的不同版本，是否够准确了？因为inode是会回收再使用的。



比如依次执行下面的命令，可以发现，两个文件的inode是一样的：



\[plain\] view plain copy

touch test  

stat test  

rm test   

touch test  

stat test  

但是因为logstash是没有close掉文件，所以是一直持有inode，所以新的同名的日志文件会有一个新的inode。



也正是因为这样，如果logstash监视的日志文件如果被删除了，还是可以继续把删除的文件的内容处理完。



利用inode这点特性，有时可以做一些补救工作，比如不小心把mysql的文件删掉了，还是可以把数据dump出来，因为mysql进程还持有数据文件的inode。



另外，logstash默认是每隔1秒就尝试读取文件有没有新内容，默认是15秒就扫描，检查有没有新文件。对应stat\_interval和discover\_interval参数。







还有一些小细节：



比如每次最多只读取出16394字节的数据，防止占用过多的内存，每5秒判断下是否需要保存新的pos。



如果日志文件被删除了，也会删除sincedb文件。



利用rename原子性地保存pos

当读取到新文件内容时，pos会增加，在保存新的pos到sincedb时，logstash采用了临时文件的办法：



先建立一个临时文件，写入新内容，再调用操作系统提供的remane函数，原子性地替换原来的sincedb文件。



这种实际上是比较常用的技巧了，redis也是这样子做的。



能否保证不重复，不丢失数据？

很遗憾，这是不能的，除非是分布式事务，否则，总有可能丢失或者重复发送数据。任何日志收集软件或者消息队列软件都是如此。



实现的代码

具体的实现代码就不贴了，因为比较易读，其中logstash使用了filewatch这个库，可以用gem来安装。相关的代码在线查看：



https://github.com/elasticsearch/logstash/blob/v1.4.1/lib/logstash/inputs/file.rb



https://github.com/jordansissel/ruby-filewatch/tree/master/lib/filewatch



和fluentd的in\_tail插件比较

fluentd也是一个很流行的日志收集工具。



简单再看了下fluentd的in\_tail插件，发现里面还有自己当年提交的一个防止内存占用过大的建议：）



https://github.com/fluent/fluentd/blob/master/lib/fluent/plugin/in\_tail.rb



                if lines.size &gt;= MAX\_LINES\_AT\_ONCE

                  \# not to use too much memory in case the file is very large

                  read\_more = true

即每最多读取1000行，就提交数据，并保存pos。

fluentd的in\_tail插件的原理和logstash的file input是差不多的，都是用inode来区分文件是否更新。



但是fluentd只保存了inode和pos，没有logstash那样把设备都考虑进去了。



另外fluentd保存pos时，都是以文件追加的方式来保存的，没有像logstash那样是用rename文件来保存到新文件里。显然logstash的实现更加合理。



扯远一点，logstash部署要比fluentd方便，尽管两者都是用ruby写的，不同的是logstash默认是jruby，只要有JVM就可以跑，fluentd则要安装ruby环境，比较麻烦。



其它的一些东东：

logstash大有一统江湖之势，这句话忘记在哪里看到的了。在github上的logstash的start有2000多个。



logstash + elasticsearch + Kibana的日志收集，搜索，展现的一条龙服务非常流行。



参考：

http://unix.stackexchange.com/questions/73988/linux-major-and-minor-device-numbers



http://zh.wikipedia.org/wiki/Inode



https://github.com/elasticsearch/logstash/blob/v1.4.1/lib/logstash/inputs/file.rb





