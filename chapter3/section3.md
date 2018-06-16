# logstash-input-jdbc实现oracle 与elasticsearch实时同步

1、配置文件

\[root@5b9dbaaa148a logstash\_jdbc\_test\]\# cat jdbc\_oracle.conf

input {

  stdin {

  }

  jdbc {

  \# oracle jdbc connection string to our backup databse

  jdbc\_connection\_string =&gt; "jdbc:oracle:thin:system/123456@//100.1.1.31:1521/xe"

  \# the user we wish to excute our statement as

  jdbc\_user =&gt; "system"

  jdbc\_password =&gt; "123456"

  \# the path to our downloaded jdbc driver

  jdbc\_driver\_library =&gt; "/elasticsearch-jdbc-2.3.2.0/lib/ojdbc6.jar"

  \# the name of the driver class for oracle

  jdbc\_driver\_class =&gt; "Java::oracle.jdbc.driver.OracleDriver"



  \#new add  begin 2016-6-28

  record\_last\_run =&gt; "true"

  use\_column\_value =&gt; "false"

  tracking\_column =&gt; "id"

  last\_run\_metadata\_path =&gt; "/etc/logstash/run\_metadata.d/my\_info"

  clean\_run =&gt; "false"

  \#new add by end



  jdbc\_paging\_enabled =&gt; "true"

  jdbc\_page\_size =&gt; "50000"

  statement\_filepath =&gt; "/usr/local/logstash/bin/logstash\_jdbc\_test/jdbc\_oracle.sql"

  schedule =&gt; "\* \* \* \* \*"

  type =&gt; "tstype"

  }

}



filter {

  json {

  source =&gt; "message"

  remove\_field =&gt; \["message"\]

  }

  \#grok {

  \#match =&gt; { "message" =&gt; "%{COMBINEDAPACHELOG}" }

  \#match =&gt; { "message" =&gt; "test" }

\#}

  date {

  match =&gt; \[ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" \]

  }

}



output {

  elasticsearch {

  hosts =&gt; "10.8.5.101:9200"

  index =&gt; "tsuser"

  document\_id =&gt; "%{user\_id}"

  }

  stdout {

  codec =&gt; json\_lines

  }

}



\[root@5b9dbaaa148a logstash\_jdbc\_test\]\# cat jdbc\_oracle.sql

select

  \*

from

  TS\_USER



2、其中:

（1） jdbc\_connection\_string配置

jdbc\_connection\_string =&gt; "jdbc:oracle:thin:system/123456@//100.1.1.31:1521/xe"

system/123456@// 100.1.1.31:1521 /xe 含义：

用户名/密码@//oracleIP地址:端口/标识符SID



（2） jdbc\_driver\_library 配置

jdbc\_driver\_library =&gt; "/elasticsearch-jdbc-2.3.2.0/lib/ojdbc6.jar"

ojdbc6.jar所在oracle机器的安装位置如下：

\[root@W01 ~\]\# find / -name "ojdbc6.jar"

/u01/app/oracle/product/11.2.0/xe/jdbc/lib/ojdbc6.jar

ojdbc6.jar 需要ES所在机器放置的位置：

\[root@5b9dbaaaa lib\]\# find / -name "ojdbc6.jar"

/elasticsearch-jdbc-2.3.2.0/lib/ojdbc6.ja



（3） jdbc\_driver\_class配置

  jdbc\_driver\_class =&gt; "Java::oracle.jdbc.driver.OracleDriver"



同步执行脚本：

\[root@5b9dbaa148a bin\]\# ./logstash -f ./logstash\_jdbc\_test/jdbc\_oracle.conf



update，insert测试ok。

 

参考详解\(原理同）：

http://blog.csdn.net/laoyang360/article/details/51747266

 

NB参考：

http://o7planning.org/en/10227/jdbc-driver-libraries-for-different-types-of-database-in-java



