# Introduction

Logstash是一款轻量级的日志搜集处理框架，可以方便的把分散的、多样化的日志搜集起来，并进行自定义的处理，然后传输到指定的位置，比如某个服务器或者文件。

下载

[https://www.elastic.co/downloads/logstash](https://www.elastic.co/downloads/logstash)

文档

[https://www.elastic.co/guide/en/logstash/current/index.html](https://www.elastic.co/guide/en/logstash/current/index.html)

下载后直接解压，就可以了。



 



　　通过命令行，进入到logstash/bin目录，执行下面的命令：



logstash -e ""

　　可以看到提示下面信息（这个命令稍后介绍），输入hello world!

![](/assets/logstashhello.png)





　　可以看到logstash尾我们自动添加了几个字段，时间戳@timestamp，版本@version，输入的类型type，以及主机名host。



工作原理

　　Logstash使用管道方式进行日志的搜集处理和输出。有点类似\*NIX系统的管道命令 xxx \| ccc \| ddd，xxx执行完了会执行ccc，然后执行ddd。



　　在logstash中，包括了三个阶段:



　　输入input --&gt; 处理filter（不是必须的） --&gt; 输出output



![](/assets/logstashfliter.png)



　　每个阶段都由很多的插件配合工作，比如file、elasticsearch、redis等等。



　　每个阶段也可以指定多种方式，比如输出既可以输出到elasticsearch中，也可以指定到stdout在控制台打印。



 



　　由于这种插件式的组织方式，使得logstash变得易于扩展和定制。



命令行中常用的命令

　　-f：通过这个命令可以指定Logstash的配置文件，根据配置文件配置logstash



![](/assets/logstash-f.png)



　　-e：后面跟着字符串，该字符串可以被当做logstash的配置（如果是“” 则默认使用stdin作为输入，stdout作为输出）



![](/assets/logstash-e.png)



　　-l：日志输出的地址（默认就是stdout直接在控制台中输出）



　　-t：测试配置文件是否正确，然后退出。



![](/assets/logstash-t.png)



配置文件说明

　　前面介绍过logstash基本上由三部分组成，input、output以及用户需要才添加的filter，因此标准的配置文件格式如下：



input {...} 

filter {...} 

output {...}

　　![](/assets/logstashconf.png)



　　在每个部分中，也可以指定多个访问方式，例如我想要指定两个日志来源文件，则可以这样写：



input { 

 file { path =&gt;"/var/log/messages" type =&gt;"syslog"} 

 file { path =&gt;"/var/log/apache/access.log" type =&gt;"apache"} 

}

　　类似的，如果在filter中添加了多种处理规则，则按照它的顺序一一处理，但是有一些插件并不是线程安全的。



　　比如在filter中指定了两个一样的的插件，这两个任务并不能保证准确的按顺序执行，因此官方也推荐避免在filter中重复使用插件。



 



　　说完这些，简单的创建一个配置文件的小例子看看：



复制代码

input {

    file {

　　　　 \#指定监听的文件路径，注意必须是绝对路径

        path =&gt; "E:/software/logstash-1.5.4/logstash-1.5.4/data/test.log"

        start\_position =&gt; beginning

    }

}

filter {

    

}

output {

    stdout {}

}

复制代码

　　日志大致如下：



1 hello,this is first line in test.log!

2 hello,my name is xingoo!

3 goodbye.this is last line in test.log!

4 

　　注意最后有一个空行。



　　执行命令得到如下信息：

![](/assets/importlogstash.png)





　　细心的会发现，这个日志输出与上面的logstash -e "" 并不相同，这是因为上面的命令默认指定了返回的格式是json形式。





