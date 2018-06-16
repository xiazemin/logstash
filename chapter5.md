# 用Fluentd实现收集日志到HDFS

Fluentd是一个实时日志收集系统，它把日志作为JSON stream，可以同时从多台server上收集大量日志，也可以构建具有层次的日志收集系统。 

Fluentd易于安装，有灵活的插件机制和缓冲，支持日志转发。它的特点在于各部分均是可定制化的，可以通过简单的配置，将日志收集到不同的地方。 

Fluentd通过hadoop中的webHDFS与HDFS进行通信，所以在配置Fluentd时，一定要保证webHDFS能正常通信。



系统环境：CentOS 6.5 

集群环境：Hadoop 2.2.0 

参考Fluentd官网。



安装

td-agent是Fluentd的一个稳定版本。 

CentOS下可以直接运行以下命令安装：



curl -L https://td-toolbelt.herokuapp.com/sh/install-redhat-td-agent2.sh \| sh

1

2

启动

管理脚本是：/etc/init.d/td-agent 

可通过/etc/init.d/td-agent start或service td-agent start来启动 

配置文件：/etc/td-agent/td-agent.conf 

重新加载配置文件：service td-agent reload 

td-agent的日志文件：/var/log/td-agent/



配置

Fluentd自带多个输入插件和输出插件，这里先实现收集本地日志到本地文件。



\#\# File input

&lt;source&gt;

  type tail

  path /var/log/mytemp.log

  pos\_file /var/log/td-agent/mytemp.log.pos

  format none

  tag td.temp

&lt;/source&gt;

\#其中：

\#1.type tail: tail方式是Fluentd内置的输入方式，其原理是不停地从源文件中获取新的日志，相当于tail –f命令。

\#2.path: 指定日志文件位置。

\#3.pos\_file：存储path中日志文件状态的文件。

\#4.format none: 指定使用何种日志解析器。

\#5.tag: tag被用来对不同的日志进行match。



\#\# File output

&lt;match td.temp&gt;

  type file

  path /var/log/td-agent/access

&lt;/match&gt;

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

当mytemp.log有更新时，更新内容会添加到access文件中。



输出到HDFS

然后我尝试将收集的日志存放到HDFS上。 

Fluentd通过webhdfs与HDFS通信，所以需要开启webhdfs。 

设置Hadoop，修改配置文件hdfs-site.xml，加入：



&lt;property&gt;

  &lt;name&gt;dfs.webhdfs.enabled&lt;/name&gt;

  &lt;value&gt;true&lt;/value&gt;

&lt;/property&gt;



&lt;property&gt;

  &lt;name&gt;dfs.support.append&lt;/name&gt;

  &lt;value&gt;true&lt;/value&gt;

&lt;/property&gt;



&lt;property&gt;

  &lt;name&gt;dfs.support.broken.append&lt;/name&gt;

  &lt;value&gt;true&lt;/value&gt;

&lt;/property&gt;

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

重启Hadoop，新建一个目录用来存放日志：



hadoop fs -mkdir /log/

hadoop fs -chmod 777 /log/

1

2

td-agent配置文件中source部分不变，修改match部分：



&lt;match td.temp&gt;

  type webhdfs

  host namenodehost

  port 50070

  path /log/a.log

  flush\_interval 5s

&lt;/match&gt;

\# flush\_interval标识数据写入HDFS的间隔



当td-agent与namenode在一台物理机上时可以正常运行，当不在一台物理机上时，报“Connection refused”错误。





