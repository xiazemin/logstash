# logstash-input-jdbc 同步原理

基于logstash-input-jdbc较其他插件的稳定性、易用性、版本和ES同步更新的特点，以下研究主要针对 logstash-input-jdbc 展开。

针对logstash-input-jdbc常见的几个疑难问题，部分问题也在git和stackoverflow进行了激烈讨论，以下统一给出验证和解答。

1、logstash-input-jdbc 的同步原理是什么？

（1）、对于全量同步依据

配置文件jdbc.sql的sql语句的进行同步。

（2）、对于增量实时同步依据

1）设定的定时策略。

如最小更新间隔每分钟更新一次设定：schedule =&gt; “\* \* \* \* \*”，目前最小更新间隔为1分钟，验证发现，不支持60s以内的秒级更新。

2）设定的sql语句。

如jdbc.sql, 决定同步哪些内容及同步更新的条件。

{"id":10,"name":"10test","@version":"1","@timestamp":"2016-06-29T03:18:00.177Z","type":"132c\_type"}

1

2：logstash-input-jdbc 只支持基于时间同步吗？

验证表名：同步更新除了支持根据时间同步外，还支持根据某自增列（如：自增ID）字段的变化进行同步。

上次举例只是举了同步时间变化的例子，设定条件：

\[root@5b9dbaaa148a logstash\_jdbc\_test\]\# cat jdbc.sql\_bak

select

```
    \*
```

from

```
    cc
```

where   cc.modified\_at &gt; :sql\_last\_value

实际进一步研究发现，在配置文件中有use\_column\_value字段决定，是否需要记录某个column 的值,如果 record\_last\_run 为真,可以自定义我们需要 track 的 column 名称，此时该参数就要为 true. 否则默认 track 的是 timestamp 的值.

举例：以下即是设定以id的变化作为同步条件的。

\[root@5b9dbaaa148a logstash\_jdbc\_test\]\# cat jdbc\_xm.sql

select

```
    \*
```

from

```
    cc
```

where   cc.id &gt;= :sql\_last\_value

我们可以指定文件,来记录上次执行到的 tracking\_column 字段的值 比如上次数据库有 12 条记录,查询完后该文件中就会有数字 12 这样的记录,下次执行 SQL 查询可以从 13 条处开始.

我们只需要在 SQL 语句中 WHERE MY\_ID &gt; :last\_sql\_value 即可. 其中 :last\_sql\_value 取得就是该文件中的值\(12\).

last\_run\_metadata\_path =&gt; “/etc/logstash/run\_metadata.d/my\_info”

如：

\[root@5b9 run\_metadata.d\]\# cat /etc/logstash/run\_metadata.d/my\_info

--- 12

已全局代码搜索，没有触发器trigger相关处理操作。

3：mysql和ES分别存储在两台服务器，且时间不一致，能否实现同步？

（1）. 设定对于以时间作为判定条件的增量同步，可以以设定的时间为基准点进行同步。

验证发现：

显示的时间戳timestamp为ES上的UTC时间值（不论ES机器是什么时区，都会修改为UTC时间存入ES），显示的modified\_at时间值为同步过来的mysql时间值转化为UTC的结果值。

更新的前提是必须满足： cc.modified\_at &gt;= :sql\_last\_value。即如果mysql的时间修改为小于sql\_last\_value的时刻值，是无法进行同步的。

如：

\[elasticsearch@5b9dbaaa148a run\_metadata.d\]$ cat my\_info

--- 2016-06-29 02:19:00.182000000 Z

（2）. 对于选定某列作为判定条件（如自增ID），两者（mysql和ES\)时间不一致，实际是也可以同步更新的。

验证发现：

测试设定的时间是mysql比ES早一天或者晚一天的时刻值，都可以实现同步更新操作。

4：如何支持实时同步mysql的delete操作到ES中？

logstash-input-jdbc插件不支持物理删除的同步更新。详见：

[http://stackoverflow.com/questions/35813923/sync-postgresql-data-with-elasticsearch/35823497\#35823497](http://stackoverflow.com/questions/35813923/sync-postgresql-data-with-elasticsearch/35823497#35823497)

[https://github.com/logstash-plugins/logstash-input-jdbc/issues/145](https://github.com/logstash-plugins/logstash-input-jdbc/issues/145)

解决方案：

同步删除操作改为同步update更新操作实现。

第一步：进行软件删除，而不是物理删除操作。

先不物理删除记录，而是软件删除，即新增一个 flag 列，标识记录是否已经被删除（默认为false，设置为true或者deleted代表已经被删除，业界通用方法），这样，通过已有的同步机制，相同的标记记录该行数据会同步更新到Elasticsearch。

第二步：ES中检索flag标记为true或者deleted的字段信息。

在ES可以执行简单的term查询操作,检索出已经删除的数据信息。

第三步：定时物理删除。

设置定时事件，间隔一段时间物理删除掉mysql和ES中的flag字段标记为true或deleted的记录，即完成物理删除操。

