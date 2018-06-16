# plugin uninstall

\[root@h102 ~\]\# /opt/logstash/bin/plugin uninstall -h 

Usage:

    bin/plugin uninstall \[OPTIONS\] PLUGIN



Parameters:

    PLUGIN                        plugin name



Options:

    -h, --help                    print help

\[root@h102 ~\]\# 

修改镜像源

\[root@h102 logstash\]\# cd /opt/logstash/

\[root@h102 logstash\]\# vim Gemfile

\[root@h102 logstash\]\# grep source Gemfile

source "https://ruby.taobao.org"

\[root@h102 logstash\]\# 

Note: 如果不修改，所有涉及插件变更的操作都会报错，原因是 The Great Wall ，如下



\[root@h102 ~\]\# /opt/logstash/bin/plugin uninstall logstash-output-kafka

Uninstalling logstash-output-kafka

Error Bundler::InstallError, retrying 1/10

An error occurred while installing arr-pm \(0.0.10\), and Bundler cannot continue.

Make sure that \`gem install arr-pm -v '0.0.10'\` succeeds before bundling.

WARNING: SSLSocket\#session= is not supported



^C

\[root@h102 ~\]\# /opt/logstash/bin/plugin install logstash-output-kafka

Validating logstash-output-kafka

Unable to download data from https://rubygems.org - Connection reset by peer \(https://rubygems.global.ssl.fastly.net/latest\_specs.4.8.gz\)

ERROR: Installation aborted, verification failed for logstash-output-kafka 

\[root@h102 ~\]\# 

即便全局的Source指的没问题



\[root@h102 ~\]\# gem source -l 

\*\*\* CURRENT SOURCES \*\*\*



https://ruby.taobao.org/

\[root@h102 ~\]\# 

但局部配置会覆盖此配置，从而实际以无法访问的地址作为自己的镜像源



\[root@h102 logstash\]\# grep source /opt/logstash/Gemfile

source "https://rubygems.org"

\[root@h102 logstash\]\# 

报错就是这么产生的



解决方法就是修改此配置到权威且可以访问的地址



详细可以参考 Private Gem Repositories



\[root@h102 ~\]\# /opt/logstash/bin/plugin list kafka 

logstash-input-kafka

logstash-output-kafka

\[root@h102 ~\]\# /opt/logstash/bin/plugin uninstall logstash-output-kafka

Uninstalling logstash-output-kafka

\[root@h102 ~\]\# /opt/logstash/bin/plugin list kafka 

logstash-input-kafka

\[root@h102 ~\]\#

plugin install

\[root@h102 ~\]\# /opt/logstash/bin/plugin install -h

Usage:

    bin/plugin install \[OPTIONS\] \[PLUGIN\] ...



Parameters:

    \[PLUGIN\] ...                  plugin name\(s\) or file



Options:

    --version VERSION             version of the plugin to install

    --\[no-\]verify                 verify plugin validity before installation \(default: true\)

    --development                 install all development dependencies of currently installed plugins \(default: false\)

    --local                       force local-only plugin installation. see bin/plugin package\|unpack \(default: false\)

    -h, --help                    print help

\[root@h102 ~\]\# /opt/logstash/bin/plugin list kafka 

logstash-input-kafka

\[root@h102 ~\]\# /opt/logstash/bin/plugin install logstash-output-kafka

Validating logstash-output-kafka

Installing logstash-output-kafka

Installation successful

\[root@h102 ~\]\# /opt/logstash/bin/plugin list kafka 

logstash-input-kafka

logstash-output-kafka

\[root@h102 ~\]\# 

plugin update

\[root@h102 ~\]\# /opt/logstash/bin/plugin update -h

Usage:

    bin/plugin update \[OPTIONS\] \[PLUGIN\] ...



Parameters:

    \[PLUGIN\] ...                  Plugin name\(s\) to upgrade to latest version



Options:

    --\[no-\]verify                 verify plugin validity before installation \(default: true\)

    --local                       force local-only plugin update. see bin/plugin package\|unpack \(default: false\)

    -h, --help                    print help

\[root@h102 ~\]\# /opt/logstash/bin/plugin update logstash-codec-edn\_lines

Updating logstash-codec-edn\_lines

No plugin updated

\[root@h102 ~\]\#

由于当前没有更新的 logstash-codec-edn\_lines ，所以没有更新



更新所有插件



\[root@h102 ~\]\# /opt/logstash/bin/plugin list --verbose 

logstash-codec-collectd \(2.0.2\)

logstash-codec-dots \(2.0.2\)

logstash-codec-edn \(2.0.2\)

logstash-codec-edn\_lines \(2.0.2\)

logstash-codec-es\_bulk \(2.0.2\)

logstash-codec-fluent \(2.0.2\)

logstash-codec-graphite \(2.0.2\)

logstash-codec-json \(2.0.4\)

logstash-codec-json\_lines \(2.0.2\)

logstash-codec-line \(2.0.2\)

logstash-codec-msgpack \(2.0.2\)

logstash-codec-multiline \(2.0.4\)

logstash-codec-netflow \(2.0.2\)

logstash-codec-oldlogstashjson \(2.0.2\)

logstash-codec-plain \(2.0.2\)

logstash-codec-rubydebug \(2.0.4\)

logstash-filter-anonymize \(2.0.2\)

logstash-filter-checksum \(2.0.2\)

logstash-filter-clone \(2.0.4\)

logstash-filter-csv \(2.1.0\)

logstash-filter-date \(2.0.2\)

logstash-filter-dns \(2.0.2\)

logstash-filter-drop \(2.0.2\)

logstash-filter-fingerprint \(2.0.2\)

logstash-filter-geoip \(2.0.4\)

logstash-filter-grok \(2.0.2\)

logstash-filter-json \(2.0.2\)

logstash-filter-kv \(2.0.2\)

logstash-filter-metrics \(3.0.0\)

logstash-filter-multiline \(2.0.3\)

logstash-filter-mutate \(2.0.2\)

logstash-filter-ruby \(2.0.2\)

logstash-filter-sleep \(2.0.2\)

logstash-filter-split \(2.0.2\)

logstash-filter-syslog\_pri \(2.0.2\)

logstash-filter-throttle \(2.0.2\)

logstash-filter-urldecode \(2.0.2\)

logstash-filter-useragent \(2.0.3\)

logstash-filter-uuid \(2.0.3\)

logstash-filter-xml \(2.0.2\)

logstash-input-beats \(2.0.3\)

logstash-input-couchdb\_changes \(2.0.2\)

logstash-input-elasticsearch \(2.0.2\)

logstash-input-eventlog \(3.0.1\)

logstash-input-exec \(2.0.4\)

logstash-input-file \(2.0.3\)

logstash-input-ganglia \(2.0.4\)

logstash-input-gelf \(2.0.2\)

logstash-input-generator \(2.0.2\)

logstash-input-graphite \(2.0.4\)

logstash-input-heartbeat \(2.0.2\)

logstash-input-http \(2.0.2\)

logstash-input-imap \(2.0.2\)

logstash-input-irc \(2.0.3\)

logstash-input-jdbc \(2.0.5\)

logstash-input-kafka \(2.0.2\)

logstash-input-log4j \(2.0.4\)

logstash-input-lumberjack \(2.0.5\)

logstash-input-pipe \(2.0.2\)

logstash-input-rabbitmq \(3.1.1\)

logstash-input-redis \(2.0.2\)

logstash-input-s3 \(2.0.3\)

logstash-input-snmptrap \(2.0.2\)

logstash-input-sqs \(2.0.3\)

logstash-input-stdin \(2.0.2\)

logstash-input-syslog \(2.0.2\)

logstash-input-tcp \(3.0.0\)

logstash-input-twitter \(2.2.0\)

logstash-input-udp \(2.0.3\)

logstash-input-unix \(2.0.4\)

logstash-input-xmpp \(2.0.3\)

logstash-input-zeromq \(2.0.2\)

logstash-output-cloudwatch \(2.0.2\)

logstash-output-csv \(2.0.2\)

logstash-output-elasticsearch \(2.2.0\)

logstash-output-email \(3.0.2\)

logstash-output-exec \(2.0.2\)

logstash-output-file \(2.2.0\)

logstash-output-ganglia \(2.0.2\)

logstash-output-gelf \(2.0.2\)

logstash-output-graphite \(2.0.2\)

logstash-output-hipchat \(3.0.2\)

logstash-output-http \(2.0.5\)

logstash-output-irc \(2.0.2\)

logstash-output-juggernaut \(2.0.2\)

logstash-output-kafka \(2.0.1\)

logstash-output-lumberjack \(2.0.4\)

logstash-output-nagios \(2.0.2\)

logstash-output-nagios\_nsca \(2.0.3\)

logstash-output-null \(2.0.2\)

logstash-output-opentsdb \(2.0.2\)

logstash-output-pagerduty \(2.0.2\)

logstash-output-pipe \(2.0.2\)

logstash-output-rabbitmq \(3.0.6\)

logstash-output-redis \(2.0.2\)

logstash-output-s3 \(2.0.3\)

logstash-output-sns \(3.0.2\)

logstash-output-sqs \(2.0.2\)

logstash-output-statsd \(2.0.4\)

logstash-output-stdout \(2.0.3\)

logstash-output-tcp \(2.0.2\)

logstash-output-udp \(2.0.2\)

logstash-output-xmpp \(2.0.2\)

logstash-output-zeromq \(2.0.2\)

logstash-patterns-core \(2.0.2\)

\[root@h102 ~\]\# /opt/logstash/bin/plugin update 

You are updating logstash-input-jdbc to a new version 3.0.0, which may not be compatible with 2.0.5. are you sure you want to proceed \(Y/N\)?

y

Updating logstash-codec-collectd, logstash-codec-dots, logstash-codec-edn, logstash-codec-edn\_lines, logstash-codec-es\_bulk, logstash-codec-fluent, logstash-codec-graphite, logstash-codec-json, logstash-codec-json\_lines, logstash-codec-line, logstash-codec-msgpack, logstash-codec-multiline, logstash-codec-netflow, logstash-codec-oldlogstashjson, logstash-codec-plain, logstash-codec-rubydebug, logstash-filter-anonymize, logstash-filter-checksum, logstash-filter-clone, logstash-filter-csv, logstash-filter-date, logstash-filter-dns, logstash-filter-drop, logstash-filter-fingerprint, logstash-filter-geoip, logstash-filter-grok, logstash-filter-json, logstash-filter-kv, logstash-filter-metrics, logstash-filter-multiline, logstash-filter-mutate, logstash-filter-ruby, logstash-filter-sleep, logstash-filter-split, logstash-filter-syslog\_pri, logstash-filter-throttle, logstash-filter-urldecode, logstash-filter-useragent, logstash-filter-uuid, logstash-filter-xml, logstash-input-beats, logstash-input-couchdb\_changes, logstash-input-elasticsearch, logstash-input-eventlog, logstash-input-exec, logstash-input-file, logstash-input-ganglia, logstash-input-gelf, logstash-input-generator, logstash-input-graphite, logstash-input-heartbeat, logstash-input-http, logstash-input-imap, logstash-input-irc, logstash-input-jdbc, logstash-input-kafka, logstash-input-log4j, logstash-input-lumberjack, logstash-input-pipe, logstash-input-rabbitmq, logstash-input-redis, logstash-input-s3, logstash-input-snmptrap, logstash-input-sqs, logstash-input-stdin, logstash-input-syslog, logstash-input-tcp, logstash-input-twitter, logstash-input-udp, logstash-input-unix, logstash-input-xmpp, logstash-input-zeromq, logstash-output-cloudwatch, logstash-output-csv, logstash-output-elasticsearch, logstash-output-email, logstash-output-exec, logstash-output-file, logstash-output-ganglia, logstash-output-gelf, logstash-output-graphite, logstash-output-hipchat, logstash-output-http, logstash-output-irc, logstash-output-juggernaut, logstash-output-kafka, logstash-output-lumberjack, logstash-output-nagios, logstash-output-nagios\_nsca, logstash-output-null, logstash-output-opentsdb, logstash-output-pagerduty, logstash-output-pipe, logstash-output-rabbitmq, logstash-output-redis, logstash-output-s3, logstash-output-sns, logstash-output-sqs, logstash-output-statsd, logstash-output-stdout, logstash-output-tcp, logstash-output-udp, logstash-output-xmpp, logstash-output-zeromq

Error Bundler::InstallError, retrying 1/10

An error occurred while installing jruby-kafka \(1.5.0\), and Bundler cannot continue.

Make sure that \`gem install jruby-kafka -v '1.5.0'\` succeeds before bundling.

WARNING: SSLSocket\#session= is not supported

Error Bundler::InstallError, retrying 2/10

An error occurred while installing manticore \(0.5.2\), and Bundler cannot continue.

Make sure that \`gem install manticore -v '0.5.2'\` succeeds before bundling.

WARNING: SSLSocket\#session= is not supported

Updated logstash-codec-json\_lines 2.0.2 to 2.0.3

Updated logstash-codec-multiline 2.0.4 to 2.0.6

Updated logstash-codec-netflow 2.0.2 to 2.0.3

\[root@h102 ~\]\# gem install jruby-kafka -v '1.5.0'

ERROR:  Could not find a valid gem 'jruby-kafka' \(= 1.5.0\) in any repository

ERROR:  Possible alternatives: jruby-kafka

\[root@h102 ~\]\# gem install manticore -v '0.5.2'

ERROR:  Could not find a valid gem 'manticore' \(= 0.5.2\) in any repository

ERROR:  Possible alternatives: MINT-core, antidote, antinode, anvil-core, fat\_core

\[root@h102 ~\]\# /opt/logstash/bin/plugin list --verbose 

logstash-codec-collectd \(2.0.2\)

logstash-codec-dots \(2.0.2\)

logstash-codec-edn \(2.0.2\)

\[root@h102 ~\]\# 

其中有几个由于找不到包，所以没有更新成功，但是大部分已经获得了更新





