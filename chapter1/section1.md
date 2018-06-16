# logstash是什么

1.logstash是什么？有什么用?

　　在网上搜索logstash，搜索结果中logstash一般是和elasticsearch、kibana一起讲的。感觉似乎logstash、elasticsearch、kibana一定要一起用，其实并不是这样的。logstash是用来收集日志用的，完全可以单独使用，只是这几个组合在一起疗效比较好。



     做过项目的都知道，传统的日志一般是直接写文件。这些日志文件里面可能写了函数调用次数、服务请求失败的日志（比如请求内容、时间、IP什么的），这些日志往往内含丰富的内容。通过分析这些日志可以让我们知道系统主要运行哪些服务、什么时候是服务的高峰期、错误比较集中的地方是哪儿，通过了解这些内容，我们可以提升系统的性能、提供更好的服务。日志文件一般是写在本地的，如果一个分布式系统的日志全部都写在本地，我们要聚合、分析这些日志也是相当麻烦的事。这时候logstash就派上用场了，日志还是原来那样写，只不过不是写文件了，而是写到RabbitMQ、Redids这些地方，然后logstash在另一头不断的取出这些日志，然后对这些日志进行一些处理，输出到另外一个地方，比如说elasticsearch或者在线分析的平台什么的。这时候日志就写到一起了，就方便使用了。



2.logstash基础

 　　在logstash中，一条条的日志其实都是事件。logstash事件中有一个个不同的字段，每个字段中都有不同的值。在logstash中，值可以有布尔、数值、字符串、数组和Hash表这5种类型。前三种就不说了，数组的写法是这样的\["hello", "world", "!"\]，Hash表的写法是这样的{key1=&gt;value1，key2=&gt;value2}。前面说到logstash就是收集日志、处理日志然后再输出出去。所以logstash就是一种input-&gt;filter-&gt;output的过程，故logstash中就有input、filter、output这三种类型的插件来处理收集到的日志。我们在配置文件中写好相关的内容、配置好插件，logstash就会按照我们所需要的那样收集、处理并输出日志。一个logstash配置文件至少包含input、output插件，filter插件根据实际需求选择。一个logstash事件的例子（瞎写的）如下：



1 {

2     message =&gt; "Hello, world!",

3     @timestamp =&gt; "2016-07-17T12:02:58.322Z"

4     host =&gt; "XXX-PC"

5 }

 



　　刚才说了logstash可以从RabbitMQ、Kafka、Redis这些消息中间件中抽取日志，这需要配置不同的input插件。如果要从某个消息中间件、文件或者其他地方收集日志，那么就需要在配置文件中配置相应的input插件。当然很多时候，实际收集到的日志和我们实际想存储的数据模型有些差别，这时候我们就可以配置filter插件来处理这些日志。一般来说单个filter插件是不能完成任务的，那我们就配置多个，然后日志就按照filter1-&gt;filter2-&gt;....-&gt;filterN这样的顺序处理。处理完了以后就通过output插件推送到不同的地方。



3.logstash运行

　　使用logstash最关键的是写好配置文件，写好配置文件了，logstash就会好好的陪你玩耍。下面举几个例子：



　　标准输入到标准输出，并且不做任何处理：



复制代码

1 input {

2     stdin {

3     }        

4 }

5 output {

6     stdout {

7         codec =&gt; rubydebug

8     }

9 }

复制代码

　　这个例子相当简单，将它保存成xxx.conf， 然后运行logstash -f xxx.conf，windows下输入 logstash.bat -f xxx.conf。你在屏幕上输入什么，就会输出一个还有message为什么的logstash事件。



　　从redis读取，输出到elasticsearch中：



复制代码

 1 input {

 2     redis {

 3         key =&gt; "logstash-\*"

 4         host =&gt; "localhost"

 5         port =&gt; 6379

 6     }

 7 }

 8 filter {

 9      kv {

10      }

11 }

12 output {

13     elasticsearch{

14         hosts =&gt; \["localhost:9200"\]

15         index =&gt; "logstash"

16         type =&gt; "logstah123"

17     }

18     stdout{

19         codec =&gt; rubydebug

20     }

21 }

复制代码

　　打开Redis客户端，以logstash-为前缀保存"key1=val1 key2=val2 key3=val3",你会在屏幕上看到key1、key2、key3都成了不同的字段，而且val1 val2 val3就是对应的值，在elasticsearch也以这种方式保存好了。这是因为kv插件可以处理这种有规则的字符串，默认以空格区分一组key-value，以等号区分key和value。可以更改kv插件的field\_split和value\_split来改变kv插件的运行表现。



4.相关链接

logstash官方文档\(内含各种插件的用法\)：



https://www.elastic.co/guide/en/logstash/current/index.html



中文书籍：



http://udn.yyuap.com/doc/logstash-best-practice-cn/get\_start/index.html





