# chapter9

nxlog

nxlog 是用 C 语言写的一个跨平台日志收集处理软件。其内部支持使用 Perl 正则和语法来进行数据结构化和逻辑判断操作。不过，其最常用的场景。是在 windows 服务器上，作为 logstash 的替代品运行。



nxlog 的 windows 安装文件下载 url 见： http://nxlog.org/system/files/products/files/1/nxlog-ce-2.8.1248.msi

heka 是 Mozilla 公司仿造 logstash 设计，用 Golang 重写的一个开源项目。同样采用了input -&gt; decoder -&gt; filter -&gt; encoder -&gt; output 的流程概念。其特点在于，在中间的 decoder/filter/encoder 部分，设计了 sandbox 概念，可以采用内嵌 lua 脚本做这一部分的工作，降低了全程使用静态 Golang 编写的难度。此外，其 filter 阶段还提供了一些监控和统计报警功能。



官网地址见：http://hekad.readthedocs.org/

