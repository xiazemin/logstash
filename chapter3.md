# Logstash Plugins

Logstash 有一套灵活的插件机制，用来方便地扩展 Logstash 的能力和特性



由于 Logstash 是使用ruby写的，所以它的插件其实就是各种gem



Logstash has a rich collection of input, filter, codec and output plugins. Plugins are available as self-contained packages called gems and hosted on RubyGems.org. The plugin manager accesed via bin/plugin script is used to manage the lifecycle of plugins in your Logstash deployment. You can install, uninstall and upgrade plugins using these Command Line Interface \(CLI\) described below.



下面分享一下 Logstash Plugins 的基础使用方法



Tip: 当前的最新版本为 Logstash 2.1.1



概要

plugin命令

获取帮助

\[root@h102 ~\]\# /opt/logstash/bin/plugin --help

Usage:

    bin/plugin \[OPTIONS\] SUBCOMMAND \[ARG\] ...



Parameters:

    SUBCOMMAND                    subcommand

    \[ARG\] ...                     subcommand arguments



Subcommands:

    install                       Install a plugin

    uninstall                     Uninstall a plugin

    update                        Update a plugin

    pack                          Package currently installed plugins

    unpack                        Unpack packaged plugins

    list                          List all installed plugins



Options:

    -h, --help                    print help

\[root@h102 ~\]\# /opt/logstash/bin/plugin list --help

Usage:

    bin/plugin list \[OPTIONS\] \[PLUGIN\]



Parameters:

    \[PLUGIN\]                      Part of plugin name to search for, leave empty for all plugins



Options:

    --installed                   List only explicitly installed plugins using bin/plugin install ... \(default: false\)

    --verbose                     Also show plugin version number \(default: false\)

    --group NAME                  Filter plugins per group: input, output, filter or codec

    -h, --help                    print help

\[root@h102 ~\]\# 

plugin list

\[root@h102 ~\]\# /opt/logstash/bin/plugin list 

logstash-codec-collectd

logstash-codec-dots

logstash-codec-edn

logstash-codec-edn\_lines

logstash-codec-es\_bulk

logstash-codec-fluent

logstash-codec-graphite

logstash-codec-json

logstash-codec-json\_lines

logstash-codec-line

logstash-codec-msgpack

logstash-codec-multiline

logstash-codec-netflow

logstash-codec-oldlogstashjson

logstash-codec-plain

logstash-codec-rubydebug

logstash-filter-anonymize

logstash-filter-checksum

logstash-filter-clone

logstash-filter-csv

logstash-filter-date

logstash-filter-dns

logstash-filter-drop

logstash-filter-fingerprint

logstash-filter-geoip

logstash-filter-grok

logstash-filter-json

logstash-filter-kv

logstash-filter-metrics

logstash-filter-multiline

logstash-filter-mutate

logstash-filter-ruby

logstash-filter-sleep

logstash-filter-split

logstash-filter-syslog\_pri

logstash-filter-throttle

logstash-filter-urldecode

logstash-filter-useragent

logstash-filter-uuid

logstash-filter-xml

logstash-input-beats

logstash-input-couchdb\_changes

logstash-input-elasticsearch

logstash-input-eventlog

logstash-input-exec

logstash-input-file

logstash-input-ganglia

logstash-input-gelf

logstash-input-generator

logstash-input-graphite

logstash-input-heartbeat

logstash-input-http

logstash-input-imap

logstash-input-irc

logstash-input-jdbc

logstash-input-kafka

logstash-input-log4j

logstash-input-lumberjack

logstash-input-pipe

logstash-input-rabbitmq

logstash-input-redis

logstash-input-s3

logstash-input-snmptrap

logstash-input-sqs

logstash-input-stdin

logstash-input-syslog

logstash-input-tcp

logstash-input-twitter

logstash-input-udp

logstash-input-unix

logstash-input-xmpp

logstash-input-zeromq

logstash-output-cloudwatch

logstash-output-csv

logstash-output-elasticsearch

logstash-output-email

logstash-output-exec

logstash-output-file

logstash-output-ganglia

logstash-output-gelf

logstash-output-graphite

logstash-output-hipchat

logstash-output-http

logstash-output-irc

logstash-output-juggernaut

logstash-output-kafka

logstash-output-lumberjack

logstash-output-nagios

logstash-output-nagios\_nsca

logstash-output-null

logstash-output-opentsdb

logstash-output-pagerduty

logstash-output-pipe

logstash-output-rabbitmq

logstash-output-redis

logstash-output-s3

logstash-output-sns

logstash-output-sqs

logstash-output-statsd

logstash-output-stdout

logstash-output-tcp

logstash-output-udp

logstash-output-xmpp

logstash-output-zeromq

logstash-patterns-core

\[root@h102 ~\]\# /opt/logstash/bin/plugin list logstash-output-kafka

logstash-output-kafka

\[root@h102 ~\]\# /opt/logstash/bin/plugin list kafka

logstash-input-kafka

logstash-output-kafka

\[root@h102 ~\]\# /opt/logstash/bin/plugin list --verbose kafka

logstash-input-kafka \(2.0.2\)

logstash-output-kafka \(2.0.1\)

\[root@h102 ~\]\# /opt/logstash/bin/plugin list --group codec

logstash-codec-collectd

logstash-codec-dots

logstash-codec-edn

logstash-codec-edn\_lines

logstash-codec-es\_bulk

logstash-codec-fluent

logstash-codec-graphite

logstash-codec-json

logstash-codec-json\_lines

logstash-codec-line

logstash-codec-msgpack

logstash-codec-multiline

logstash-codec-netflow

logstash-codec-oldlogstashjson

logstash-codec-plain

logstash-codec-rubydebug

\[root@h102 ~\]\# 



