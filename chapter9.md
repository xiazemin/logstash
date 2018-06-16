# chapter9

nxlog

nxlog 是用 C 语言写的一个跨平台日志收集处理软件。其内部支持使用 Perl 正则和语法来进行数据结构化和逻辑判断操作。不过，其最常用的场景。是在 windows 服务器上，作为 logstash 的替代品运行。

nxlog 的 windows 安装文件下载 url 见： [http://nxlog.org/system/files/products/files/1/nxlog-ce-2.8.1248.msi](http://nxlog.org/system/files/products/files/1/nxlog-ce-2.8.1248.msi)

heka 是 Mozilla 公司仿造 logstash 设计，用 Golang 重写的一个开源项目。同样采用了input -&gt; decoder -&gt; filter -&gt; encoder -&gt; output 的流程概念。其特点在于，在中间的 decoder/filter/encoder 部分，设计了 sandbox 概念，可以采用内嵌 lua 脚本做这一部分的工作，降低了全程使用静态 Golang 编写的难度。此外，其 filter 阶段还提供了一些监控和统计报警功能。

官网地址见：[http://hekad.readthedocs.org/](http://hekad.readthedocs.org/)

Message::Passing 是一个仿造 Logstash 写的 Perl5 项目。项目早期甚至就直接照原样也叫 "Logstash" 的名字。



但实际上，Message::Passing 内部原理设计还是有所偏差的。这个项目整个基于 AnyEvent 事件驱动开发框架\(即著名的 libev 库\)完成，也要求所有插件不要采取阻塞式编程。所以，虽然项目开发不太活跃，插件接口不甚完善，但是性能方面却非常棒。这也是我从多个 Perl 日志处理框架中选择介绍这个的原因。



Message::Passing 有比较全的 input 和 output 插件，这意味着它可以通过多种协议跟 logstash 混跑，不过 filter 插件比较缺乏。对等于 grok 的插件叫 Message::Passing::Filter::Regexp\( 我写的，嘿嘿\)。下面是一个完整的配置示例：

fluent

