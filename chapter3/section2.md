# plugin pack

\[root@h102 ~\]\# /opt/logstash/bin/plugin pack -h

Usage:

    bin/plugin pack \[OPTIONS\]



Options:

    --tgz                         compress package as a tar.gz file \(default: true\)

    --zip                         compress package as a zip file \(default: false\)

    --\[no-\]clean                  clean up the generated dump of plugins \(default: true\)

    --overwrite                   Overwrite a previously generated package file \(default: false\)

    -h, --help                    print help

\[root@h102 ~\]\# /opt/logstash/bin/plugin pack   --tgz  

Packaging plugins for offline usage

Generated at /opt/logstash/plugins\_package.tar.gz

\[root@h102 ~\]\# ll -h /opt/logstash/plugins\_package.tar.gz

-rw-r--r-- 1 root root 52M Jan  8 16:26 /opt/logstash/plugins\_package.tar.gz

\[root@h102 ~\]\# 

plugin unpack

\[root@h102 ~\]\# /opt/logstash/bin/plugin unpack -h

Usage:

    bin/plugin unpack \[OPTIONS\] file



Parameters:

    file                          the package file name



Options:

    --tgz                         unpack a packaged tar.gz file \(default: true\)

    --zip                         unpack a packaged  zip file \(default: false\)

    -h, --help                    print help

\[root@h102 ~\]\# /opt/logstash/bin/plugin unpack --tgz /opt/logstash/plugins\_package.tar.gz

Unpacking /opt/logstash/plugins\_package.tar.gz

Unpacked at /opt/logstash/vendor/cache

The unpacked plugins can now be installed in local-only mode using bin/plugin install --local \[plugin name\]

\[root@h102 ~\]\# echo $?

0

\[root@h102 ~\]\# 

pack/unpack 主要是用来进行离线管理 plugins



\[root@h102 ~\]\# /opt/logstash/bin/plugin uninstall logstash-input-twitter

Uninstalling logstash-input-twitter

\[root@h102 ~\]\# /opt/logstash/bin/plugin list twitter

ERROR: No plugins found

\[root@h102 ~\]\# /opt/logstash/bin/plugin install --local logstash-input-twitter

Installing logstash-input-twitter

Installation successful

\[root@h102 ~\]\# /opt/logstash/bin/plugin list twitter

logstash-input-twitter

\[root@h102 ~\]\#

其它plugin

其它plugin可以参考下面链接



Input Plugins



Filter Plugins



Output Plugins



Codec Plugins





