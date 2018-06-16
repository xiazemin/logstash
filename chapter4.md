# logstash的logstash-input-kafka插件读取kafka中的数据

logstash版本为5.5.3，kafka版本为2.11，此版本默认内置了kafka插件，可直接配置使用，不需要重新安装插件；注意logstash5.x版本前后配置不太一样，注意甄别，必要时可去elasticsearch官网查看最新版配置参数的变化，例如logstash5.x版本以前kafka插件配置的是zookeeper地址，5.x以后配置的是kafka实例地址。



input{

      kafka{

        bootstrap\_servers =&gt; \["192.168.110.31:9092,192.168.110.31:9093,192.168.110.31:9094"\]

        client\_id =&gt; "test"

        group\_id =&gt; "test"

        auto\_offset\_reset =&gt; "latest" //从最新的偏移量开始消费

        consumer\_threads =&gt; 5

        decorate\_events =&gt; true //此属性会将当前topic、offset、group、partition等信息也带到message中

        topics =&gt; \["logq","loge"\] //数组类型，可配置多个topic

        type =&gt; "bhy" //所有插件通用属性,尤其在input里面配置多个数据源时很有用

      }

}

    使用了decorate\_events属性，注意看logstash控制台打印的信息，会输出如下



"kafka":{"consumer\_group":"test","partition":0,"offset":10430232,"topic":"logq","key":null}

   另外一个input里面可设置多个kafka，

input{

      kafka{

        bootstrap\_servers =&gt; \["192.168.110.31:9092,192.168.110.31:9093,192.168.110.31:9094"\]

        client\_id =&gt; "test1"

        group\_id =&gt; "test1"

        auto\_offset\_reset =&gt; "latest"

        consumer\_threads =&gt; 5

        decorate\_events =&gt; true

        topics =&gt; \["loge"\]

        type =&gt; "classroom"

      }

      kafka{

        bootstrap\_servers =&gt; \["192.168.110.31:9092,192.168.110.31:9093,192.168.110.31:9094"\]

        client\_id =&gt; "test2"

        group\_id =&gt; "test2"

        auto\_offset\_reset =&gt; "latest"

        consumer\_threads =&gt; 5

        decorate\_events =&gt; true

        topics =&gt; \["logq"\]

        type =&gt; "student"

      }

}

        假如你在filter模块中还要做其他过滤操作，并且针对input里面的每个数据源做得操作不一样，那你就可以根据各自定义的type来匹配

filter{

        if\[type\] == "classroom"{

            grok{

               ........

            }

        }

        if\[type\] == "student"{

            mutate{

               ........

            }

        }

}

         不只filter中可以这样，output里面也可以这样；并且当output为elasticsearch的时候，input里面的定义的type将会成为elasticsearch的你定义的index下的type

output {

        if\[type\] == "classroom"{

          elasticsearch{

               hosts =&gt; \["192.168.110.31:9200"\]

               index =&gt; "school"

               timeout =&gt; 300

               user =&gt; "elastic"

               password =&gt; "changeme"

          }



        }

        if\[type\] == "student"{

            ........

        }

 }

         对于第一个存储到elasticsearch的路径为localhost:9200/school/classroom；第二个存储到elasticsearch的路径为localhost:9200/school/student。假如从来没有定义过type，默认的type为logs，访问路径为第一个存储到elasticsearch的路径为localhost:9200/school/logs，默认的type也可不加





