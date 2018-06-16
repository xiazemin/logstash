# ELK 搭建



ELK是三个开源软件的缩写，分别表示：Elasticsearch , Logstash, Kibana , 它们都是开源软件



Elasticsearch 是一个分布式搜索分析引擎，稳定、可水平扩展、易于管理是它的主要设计初衷



Logstash 是一个灵活的数据收集、加工和传输的管道软件



Kibana 是一个数据可视化平台，可以通过将数据转化为酷炫而强大的图像而实现与数据的交互



将三者的收集加工，存储分析和可视转化整合在一起就形成了 ELK



相关产品的详细信息可以参考 官网 ，下面分享一下 ELK 的搭建与基础操作，详细可以参阅 官方文档



Tip: 三个软件当前的最新版本分别为 Logstash 2.1.1 , Kibana 4.3.1 , Elasticsearch 2.1.1



概要

依赖

Elasticsearch requires at least Java 7

Elasticsearch requires at least Java 7. Specifically as of this writing, it is recommended that you use the Oracle JDK version 1.8.0\_25. Java installation varies from platform to platform so we won’t go into those details here. Oracle’s recommended installation documentation can be found on Oracle’s website. Suffice to say, before you install Elasticsearch, please check your Java version first by running \(and then install/upgrade accordingly if needed\)



Logstash requires Java 7

Logstash requires Java 7 or later. Use the official Oracle distribution or an open-source distribution such as OpenJDK.



Kibana 4.3.1 requires Elasticsearch 2.1

检查java版本

\[root@h102 ELK\]\# java -version

java version "1.7.0\_65"

OpenJDK Runtime Environment \(rhel-2.5.1.2.el6\_5-x86\_64 u65-b17\)

OpenJDK 64-Bit Server VM \(build 24.65-b04, mixed mode\)

\[root@h102 ELK\]\# 

符合要求



下载

elastic 所有软件的 下载地址



创建仓库

\[root@h102 ELK\]\# vim /etc/yum.repos.d/elk.repo 

\[root@h102 ELK\]\# cat /etc/yum.repos.d/elk.repo 

\[logstash-2.1\]

name=Logstash repository for 2.1.x packages

baseurl=http://packages.elastic.co/logstash/2.1/centos

gpgcheck=1

gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch

enabled=1

\[elasticsearch-2.x\]

name=Elasticsearch repository for 2.x packages

baseurl=http://packages.elastic.co/elasticsearch/2.x/centos

gpgcheck=1

gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch

enabled=1

\[root@h102 ELK\]\# 

导入GPG校验密钥

\[root@h102 ELK\]\# wget  https://packages.elastic.co/GPG-KEY-elasticsearch

--2015-12-22 17:40:51--  https://packages.elastic.co/GPG-KEY-elasticsearch

Resolving packages.elastic.co... 50.16.251.137, 50.17.224.164, 23.21.65.76, ...

Connecting to packages.elastic.co\|50.16.251.137\|:443... connected.

HTTP request sent, awaiting response... 200 OK

Length: 1768 \(1.7K\) \[binary/octet-stream\]

Saving to: “GPG-KEY-elasticsearch”



100%\[======================================&gt;\] 1,768       --.-K/s   in 0s      



2015-12-22 17:41:03 \(143 MB/s\) - “GPG-KEY-elasticsearch” saved \[1768/1768\]



\[root@h102 ELK\]\# rpm --import GPG-KEY-elasticsearch 

\[root@h102 ELK\]\# echo $?

0

\[root@h102 ELK\]\#

下载软件

\[root@h102 ELK\]\# yumdownloader elasticsearch logstash 

Loaded plugins: dellsysid, fastestmirror, refresh-packagekit

Loading mirror speeds from cached hostfile

 \* base: mirrors.opencas.cn

 \* epel: mirrors.opencas.cn

 \* extras: ftp.sjtu.edu.cn

 \* updates: ftp.sjtu.edu.cn

elasticsearch-2.1.1.rpm                                                                                        \|  28 MB     01:39  

logstash-2.1.1-1.noarch.rpm                                                                                    \|  71 MB     02:50      

\[root@h102 ELK\]\# wget https://download.elastic.co/kibana/kibana/kibana-4.3.1-linux-x64.tar.gz

--2015-12-22 17:42:45--  https://download.elastic.co/kibana/kibana/kibana-4.3.1-linux-x64.tar.gz

Resolving download.elastic.co... 50.16.251.137, 50.17.224.164, 23.21.65.76, ...

Connecting to download.elastic.co\|50.16.251.137\|:443... connected.

HTTP request sent, awaiting response... 200 OK

Length: 30408272 \(29M\) \[application/octet-stream\]

Saving to: “kibana-4.3.1-linux-x64.tar.gz”



100%\[============================================================================================&gt;\] 30,408,272  15.6K/s   in 20m 39s 



2015-12-22 18:03:36 \(24.0 KB/s\) - “kibana-4.3.1-linux-x64.tar.gz” saved \[30408272/30408272\]



\[root@h102 ELK\]\# 

进行校验

\[root@h102 ELK\]\# sha1sum \* 

c2b6831386d926ad29f0e1abfcb8ae11f5505084  elasticsearch-2.1.1.rpm

84462fee86fc70185a9e83da42e78c2d57ef0985  GPG-KEY-elasticsearch

115ba22882df75eb5f07330b7ad8781a57569b00  kibana-4.3.1-linux-x64.tar.gz

a72ccab73566e52e61d6dcedf14c14e10257b243  logstash-2.1.1-1.noarch.rpm

\[root@h102 ELK\]\# 

Tip: 之所以选择下载，而不是直接安装是为了避免重复安装的重复下载问题，这几个包都比较大，相对比较耗费带宽，网络如出问题，也容易前功尽弃



安装

安装elasticsearch

\[root@h102 ELK\]\# rpm -ivh elasticsearch-2.1.1.rpm  

Preparing...                \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# \[100%\]

Creating elasticsearch group... OK

Creating elasticsearch user... OK

   1:elasticsearch          \#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\# \[100%\]

\#\#\# NOT starting on installation, please execute the following statements to configure elasticsearch service to start automatically using chkconfig

 sudo chkconfig --add elasticsearch

\#\#\# You can start elasticsearch service by executing

 sudo service elasticsearch start

\[root@h102 ELK\]\# chkconfig --add elasticsearch

\[root@h102 ELK\]\# chkconfig  --list \| grep elasticsearch

elasticsearch  	0:off	1:off	2:on	3:on	4:on	5:on	6:off 

\[root@h102 ELK\]\# /etc/init.d/elasticsearch start 

Starting elasticsearch:                                    \[  OK  \]

\[root@h102 ELK\]\# ps faux \| grep java 

root      8794  0.0  0.0 103256   828 pts/1    S+   19:31   0:00          \\_ grep java

488       8738 91.3 12.4 2526348 238520 ?      Sl   19:31   0:21 /usr/bin/java -Xms256m -Xmx1g -Djava.awt.headless=true -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+HeapDumpOnOutOfMemoryError -XX:+DisableExplicitGC -Dfile.encoding=UTF-8 -Djna.nosys=true -Des.path.home=/usr/share/elasticsearch -cp /usr/share/elasticsearch/lib/elasticsearch-2.1.1.jar:/usr/share/elasticsearch/lib/\* org.elasticsearch.bootstrap.Elasticsearch start -p /var/run/elasticsearch/elasticsearch.pid -d -Des.default.path.home=/usr/share/elasticsearch -Des.default.path.logs=/var/log/elasticsearch -Des.default.path.data=/var/lib/elasticsearch -Des.default.path.conf=/etc/elasticsearch

\[root@h102 ELK\]\# netstat  -ant \| grep 9200

tcp        0      0 ::1:9200                    :::\*                        LISTEN      

tcp        0      0 ::ffff:127.0.0.1:9200       :::\*                        LISTEN       

\[root@h102 ELK\]\# netstat  -ant \| grep 9300

tcp        0      0 ::1:9300                    :::\*                        LISTEN      

tcp        0      0 ::ffff:127.0.0.1:9300       :::\*                        LISTEN      

\[root@h102 ELK\]\# curl localhost:9200/\_cat/health?v

epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending\_tasks max\_task\_wait\_time active\_shards\_percent 

1450783966 19:32:46  elasticsearch green           1         1      0   0    0    0        0             0                  -                100.0% 

\[root@h102 ELK\]\# curl localhost:9200/\_cat/nodes?v

host      ip        heap.percent ram.percent load node.role master name 

127.0.0.1 127.0.0.1           12          70 0.09 d         \*      Pyre 

\[root@h102 ELK\]\# curl 'localhost:9200/\_cat/allocation?v'

shards disk.indices disk.used disk.avail disk.total disk.percent host      ip        node 

     0           0b    24.9gb     24.1gb       49gb           50 127.0.0.1 127.0.0.1 Pyre 

\[root@h102 ELK\]\#

安装kibana

准确来说，只用解压就可以了



\[root@h102 ELK\]\# ls

elasticsearch-2.1.1.rpm  GPG-KEY-elasticsearch  kibana-4.3.1-linux-x64.tar.gz  logstash-2.1.1-1.noarch.rpm

\[root@h102 ELK\]\# tar -zxvf kibana-4.3.1-linux-x64.tar.gz 

kibana-4.3.1-linux-x64/

kibana-4.3.1-linux-x64/bin/

kibana-4.3.1-linux-x64/config/

kibana-4.3.1-linux-x64/installedPlugins/

kibana-4.3.1-linux-x64/LICENSE.txt

kibana-4.3.1-linux-x64/node/

kibana-4.3.1-linux-x64/node\_modules/

kibana-4.3.1-linux-x64/optimize/

kibana-4.3.1-linux-x64/package.json

kibana-4.3.1-linux-x64/README.txt

kibana-4.3.1-linux-x64/src/

...

...

kibana-4.3.1-linux-x64/node/bin/node

kibana-4.3.1-linux-x64/node/bin/npm

kibana-4.3.1-linux-x64/config/kibana.yml

kibana-4.3.1-linux-x64/bin/kibana

kibana-4.3.1-linux-x64/bin/kibana.bat

\[root@h102 ELK\]\# ls

elasticsearch-2.1.1.rpm  GPG-KEY-elasticsearch  kibana-4.3.1-linux-x64  kibana-4.3.1-linux-x64.tar.gz  logstash-2.1.1-1.noarch.rpm

\[root@h102 ELK\]\# cd kibana-4.3.1-linux-x64/config/

\[root@h102 config\]\# ls

kibana.yml

\[root@h102 config\]\# grep -v "^\#" kibana.yml  \| grep -v "^$"

\[root@h102 config\]\# vim kibana.yml 

\[root@h102 config\]\# grep -v "^\#" kibana.yml  \| grep -v "^$"

elasticsearch.url: "http://localhost:9200"

\[root@h102 config\]\# cd ../bin/

\[root@h102 bin\]\# ls

kibana  kibana.bat

\[root@h102 bin\]\# ./kibana  

  log   \[19:56:19.616\] \[info\]\[status\]\[plugin:kibana\] Status changed from uninitialized to green - Ready

  log   \[19:56:19.956\] \[info\]\[status\]\[plugin:elasticsearch\] Status changed from uninitialized to yellow - Waiting for Elasticsearch

  log   \[19:56:20.002\] \[info\]\[status\]\[plugin:kbn\_vislib\_vis\_types\] Status changed from uninitialized to green - Ready

  log   \[19:56:20.054\] \[info\]\[status\]\[plugin:markdown\_vis\] Status changed from uninitialized to green - Ready

  log   \[19:56:20.074\] \[info\]\[status\]\[plugin:metric\_vis\] Status changed from uninitialized to green - Ready

  log   \[19:56:20.137\] \[info\]\[status\]\[plugin:spyModes\] Status changed from uninitialized to green - Ready

  log   \[19:56:20.163\] \[info\]\[status\]\[plugin:statusPage\] Status changed from uninitialized to green - Ready

  log   \[19:56:20.179\] \[info\]\[status\]\[plugin:table\_vis\] Status changed from uninitialized to green - Ready

  log   \[19:56:20.269\] \[info\]\[listening\] Server running at http://0.0.0.0:5601

  log   \[19:56:25.445\] \[info\]\[status\]\[plugin:elasticsearch\] Status changed from yellow to yellow - No existing Kibana index found

  log   \[19:56:34.094\] \[info\]\[status\]\[plugin:elasticsearch\] Status changed from yellow to green - Kibana index ready

  log   \[19:58:01.906\] \[error\]\[status\]\[plugin:elasticsearch\] Status changed from green to red - Request Timeout after 1500ms

  log   \[19:58:04.434\] \[info\]\[status\]\[plugin:elasticsearch\] Status changed from red to green - Kibana index ready

...

...

配置目录和命令目录



\[root@h102 kibana-4.3.1-linux-x64\]\# tree bin/ config/

bin/

├── kibana

└── kibana.bat

config/

└── kibana.yml



0 directories, 3 files

\[root@h102 kibana-4.3.1-linux-x64\]\#

配置很简单，只用解注 config/kibana.yml 中这一行 elasticsearch.url: “http://localhost:9200”



启动也很简单，只用执行 bin/kibana 就可以在前台运行



由于 kibana 启动后默认会开启 5601 端口，所以如果想从外面访问，得开启防火墙



开启防火墙

\[root@h102 ~\]\# grep 5601 /etc/sysconfig/iptables

\[root@h102 ~\]\# vim /etc/sysconfig/iptables

\[root@h102 ~\]\# grep 5601 /etc/sysconfig/iptables

-A INPUT -p tcp -m state --state NEW -m tcp --dport 5601 -j ACCEPT 

\[root@h102 ~\]\# iptables -L -nv \| grep 5601

\[root@h102 ~\]\# /etc/init.d/iptables  reload 

iptables: Trying to reload firewall rules:                 \[  OK  \]

\[root@h102 ~\]\# iptables -L -nv \| grep 5601

    0     0 ACCEPT     tcp  --  \*      \*       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:5601 

\[root@h102 ~\]\# 

此时已经可以进行访问了

