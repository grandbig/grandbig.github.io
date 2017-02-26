---
layout: post
title: "Dockerコンテナ内のUbuntuでRedis設定を試してみよう！"
date: 2017-02-25 22:36
comments: true
categories: docker redis
---

### はじめに
今回はDockerコンテナ内のUbuntuに[Redis](https://redis.io/)をインストールしてみたいと思います。  
基本的な操作だけでなく、デフォルトの設定とそれらの意味についても見ていきます。  

### UbuntuへのRedisのインストール
まずはインストールをしないと始まりません。  
その方法について見ていきます。  
今回扱う環境は以下の通りです。  

* Ubuntu: 14.04.5 LTS  
* Redis: 2.8.4  

バージョン的に最新ではない部分もあるのですが、Dockerコンテナ内のサーバを構築する上で利用したバージョンで進めます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

インストールは実に簡単です。  
`apt-get`を利用するのみです。  

```javascript
$ apt-get install redis-server
Reading package lists... Done
Building dependency tree       
Reading state information... Done
...
Setting up redis-server (2:2.8.4-2) ...
Starting redis-server: redis-server.
Processing triggers for libc-bin (2.19-0ubuntu6.9) ...
Processing triggers for ureadahead (0.100.0-16) ...
```

上記のログで`Starting redis-server: redis-server.`として出力されている通り、インストールが完了すると、Redisを起動してくれます。  
念のため確認してみますが、  

```javascript
$ ps aux | grep redis
redis     8833  0.0  0.1  37000  3348 ?        Ssl  13:25   0:00 /usr/bin/redis-server 127.0.0.1:6379       
root      8843  0.0  0.0   8868   784 ?        S+   13:26   0:00 grep --color=auto redis
```

ご覧の通り起動しています。  

### Redisの起動
インストール後、自動で起動してくれているものの、手動での起動と停止状態は知っておく必要があります。  

```javascript
// Redisの起動(続けて操作するためにBG起動)
$ redis-server &

// Redisの停止
$ redis-cli shutdown
```

### Redisの基本コマンド
Redisの基本コマンドは以下の通りです。  

* `keys *`: 現時点で保存されているkey全てを表示  
* `set key value`: keyとvalueの格納    
* `get key`: 指定したkeyのvalueを取得  
* `del key`: 指定したkeyのvalueを削除  
* `flushdb`: 現時点で保存されているkey全てを削除  

実際に使ってみましょう。  

```javascript
$ redis-cli
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> set user takahiro
OK
127.0.0.1:6379> set device iphone
OK
127.0.0.1:6379> keys *
1) "user"
2) "device"
127.0.0.1:6379> get user
"takahiro"
127.0.0.1:6379> get device
"iphone"
127.0.0.1:6379> del device
(integer) 1
127.0.0.1:6379> keys *
1) "user"
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> keys *
(empty list or set)
```

### Redisのデフォルト設定
続いて、Redisの設定を見ていきます。  
インストール後、`redis.conf`ファイルが`/etc/redis/`配下に作成されます。  
デフォルトでこのファイルの中身には多くのことが書かれています。  
1つずつ見ていきましょう。  

#### 単位指定について
まずはメモリ単位関連の説明です。  

```javascript
# Redis configuration file example

# Note on units: when memory size is needed, it is possible to specify
# it in the usual form of 1k 5GB 4M and so forth:
#
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same.
```

もう上記のままですね。  
`1k`か`1kb`なのか微妙に書き方を気をつけないと想定外の容量指定になるので気をつけましょう。  
ただし、`1GB` / `1Gb` / `1gB`は微妙に書き方が異なりますが指定容量は同じです。  

#### includeによる設定ファイルの分割について
Apacheなどでもよくある通り`include`を利用することもできます。  

```javascript
################################## INCLUDES ###################################

# Include one or more other config files here.  This is useful if you
# have a standard template that goes to all Redis server but also need
# to customize a few per-server settings.  Include files can include
# other files, so use this wisely.
#
# Notice option "include" won‘t be rewritten by command "CONFIG REWRITE"
# from admin or Redis Sentinel. Since Redis always uses the last processed
# line as value of a configuration directive, you‘d better put includes
# at the beginning of this file to avoid overwriting config change at runtime.
#
# If instead you are interested in using includes to override configuration
# options, it is better to use include as the last line.
#
# include /path/to/local.conf
# include /path/to/other.conf
```

サーバごとに設定を分けたいとか、第三者が見やすいように設定単位でファイルを分けたいとか様々な用途で便利に利用できるはずです。  
`include`したファイルに定義した内容で`redis.conf`の定義内容を書き換えるのであれば、`redis.conf`の一番最後に`include`を書いた方が良いでしょう。  
(逆に、書き換えたくないのであれば、`redis.conf`の初めに書きましょう。)  

#### 一般設定について
さて、最も基本となる設定です。  
少々、長いので細かめに分割してみていきましょう。  

##### デーモン化の有無
デーモン化の有無は下記で設定します。  

```javascript
# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize yes
```

デフォルト設定ではデーモン起動はしないようになっています。  
* yes: デーモン起動する  
* no: デーモン起動しない  

PIDファイルはデフォルトで`/var/run/redis.pid`に出力します。  

##### PIDファイルの指定
デフォルトとは異なる場所にPIDファイルを出力したい場合に指定します。  

```javascript
# When running daemonized, Redis writes a pid file in /var/run/redis.pid by
# default. You can specify a custom pid file location here.
pidfile /var/run/redis/redis-server.pid
```

##### ボート番号の指定
Redisを起動する際のポート番号を指定します。  

```javascript
# Accept connections on the specified port, default is 6379.
# If port 0 is specified Redis will not listen on a TCP socket.
port 6379
```

* デフォルトポート番号: 6379  
* 0指定: TCPソケット通信を受け付けない  

##### IPアドレスの指定
Redisを起動する際の接続するIPアドレスを指定します。  

```javascript
# By default Redis listens for connections from all the network interfaces
# available on the server. It is possible to listen to just one or multiple
# interfaces using the "bind" configuration directive, followed by one or
# more IP addresses.
#
# Examples:
#
# bind 192.168.1.100 10.0.0.1
bind 127.0.0.1
```

複数のIPアドレスを指定する場合は半角スペースを挟んで連続して書きましょう。  

##### Unixドメインソケット通信の設定
TCP通信だけでなく、Unixドメインソケット通信を受け付ける場合に設定します。  

```javascript
# Specify the path for the unix socket that will be used to listen for
# incoming connections. There is no default, so Redis will not listen
# on a unix socket when not specified.
#
# unixsocket /var/run/redis/redis.sock
# unixsocketperm 755
```

* デフォルトではコメントアウトされている  
  * 同一サーバ内での受付はTCPより良いという話も...!?  
* `unixsocket`: `socket`ファイルのパス指定  
* `unixsocketperm`: `socket`ファイルへのアクセス権  

##### タイムアウトの設定
クライアントからの接続タイムアウト設定です。  

```javascript
# Close the connection after a client is idle for N seconds (0 to disable)
timeout 0
```

0を設定すると、タイムアウトは無効になります。  

##### TCP KeepAlive設定
TCP通信の生存確認に関する設定です。  

```javascript
# TCP keepalive.
#
# If non-zero, use SO_KEEPALIVE to send TCP ACKs to clients in absence
# of communication. This is useful for two reasons:
#
# 1) Detect dead peers.
# 2) Take the connection alive from the point of view of network
#    equipment in the middle.
#
# On Linux, the specified value (in seconds) is the period used to send ACKs.
# Note that to close the connection the double of the time is needed.
# On other kernels the period depends on the kernel configuration.
#
# A reasonable value for this option is 60 seconds.
tcp-keepalive 0
```

* 有効にすることで、通信断(Dead Peer)状態を防いだり、コネクションの接続生存を確認することができます  
* 適切な値として `60秒` が推奨されています  

##### ログレベルの設定
ログの出力量を調整するために設定します。  

```javascript
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
loglevel notice
```

* `debug`: 開発やテスト用に利用。大量のログを出力します  
* `verbose`: `debug`よりは少ないものの多くのログを出力します  
* `notice`: 商用に適した設定です  
* `warning`: 重要またはクリティカルなメッセージのみログに出します  

##### ログファイルの出力先の指定
ログファイルの出力先のパスを指定します。  

```javascript
# Specify the log file name. Also the empty string can be used to force
# Redis to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
logfile /var/log/redis/redis-server.log
```

##### システムログの設定
システムログに関する設定です。  

```javascript
# To enable logging to the system logger, just set 'syslog-enabled' to yes,
# and optionally update the other syslog parameters to suit your needs.
# syslog-enabled no

# Specify the syslog identity.
# syslog-ident redis

# Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.
# syslog-facility local0
```

* `syslog-enabled`: システムログのロギングの有無を設定します  
* `syslog-ident`: システムログの識別子の設定をします  
* `syslog-facility`: システムログの分類を設定します  
  * `user`: ユーザプロセスのメッセージを記録するために設定します  
  * `local0 〜 7`: セキュリティやネットワークなど細かく設定できます  

`local0 〜 7`については[syslog facility](https://www.furukawa.co.jp/fitelnet/f/man/100/command-config/global_config/syslog_facility.htm#facility)が参考になりそうです。  

##### データベースの設定
使用するデータベース番号を指定します。  
デフォルトで`16`が指定されていることで利用可能なデータベース番号は`0 〜 15`となっています。  
特に何もしなければ`DB 0`を利用します。  
`SELECT`文を利用することでデータベースを指定して使うこともできます。  

```javascript
# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16
```

#### スナップショット(バックアップ)設定
こちらもかなり重要な設定となります。  
利用用途にもよりますが、サービス上、消えてしまってはまずいデータを扱うのであれば堅牢な仕組みになるように設定を考える必要があります。  

##### ディスク保存の頻度設定
`key`の変更数と保存までに時間を設定します。  
これにより、ある程度自由に堅牢性を設定することが可能です。  

```javascript
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving at all commenting all the "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""

save 900 1
save 300 10
save 60 10000
```

上記デフォルト設定の意味は下記です。  
* `save 900 1`: 1個の`key`が変更されたら、15分後にディスクにその状態を保存  
* `sabe 300 10`: 10個の`key`が変更されたら、5分後にディスクにその状態を保存  
* `save 60 10000`: 10000個の`key`が変更されたら、1分後にディスクにその状態を保存  

もしディスク保存したくなければ(インメモリでの利用のみであれば)`save ""`を指定しましょう。  

##### エラー発生時の保存挙動の設定
これはエラーが発生したときにディスク保存をどうするかを決める設定です。  

```javascript
# By default Redis will stop accepting writes if RDB snapshots are enabled
# (at least one save point) and the latest background save failed.
# This will make the user aware (in a hard way) that data is not persisting
# on disk properly, otherwise chances are that no one will notice and some
# disaster will happen.
#
# If the background saving process will start working again Redis will
# automatically allow writes again.
#
# However if you have setup your proper monitoring of the Redis server
# and persistence, you may want to disable this feature so that Redis will
# continue to work as usual even if there are problems with disk,
# permissions, and so forth.
stop-writes-on-bgsave-error yes
```

* yes: 保存を止めます  
* no: 書き込みを続けます  

##### Dump時の圧縮設定
データをDumpする際に`LZF`圧縮をかけるか否かを設定できます。  

```javascript
# Compress string objects using LZF when dump .rdb databases?
# For default that‘s set to 'yes' as it‘s almost always a win.
# If you want to save some CPU in the saving child set it to 'no' but
# the dataset will likely be bigger if you have compressible values or keys.
rdbcompression yes
```

* yes: 圧縮します(CPU負荷が上がる一方で小容量)  
* no: 圧縮しません(CPU負荷が下がる一方で大容量)  

##### Checksum設定
これはデータの整合性を検証するための設定です。  

```javascript
# Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
# This makes the format more resistant to corruption but there is a performance
# hit to pay (around 10%) when saving and loading RDB files, so you can disable it
# for maximum performances.
#
# RDB files created with checksum disabled have a checksum of zero that will
# tell the loading code to skip the check.
rdbchecksum yes
```

データ保存時にRDBファイルの読み込みを行うため、10%の性能劣化が発生するが、堅牢なデータを担保したいのであれば設定して損はないでしょう。  

##### Dumpファイル名の設定
Dumpしたファイルの名称を指定します。  

```javascript
# The filename where to dump the DB
dbfilename dump.rdb
```

##### Dumpファイルの保存場所の設定
どこにDumpファイルを配置するかの設定です。  

```javascript
# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir /var/lib/redis
```

####レプリケーション設定
障害発生時に利用するレプリケーションの設定についてもデフォルトで説明書きがなされています。  

##### スレーブ設定
スレーブのコピー元となるマスターを指定します。  

```javascript
# Master-Slave replication. Use slaveof to make a Redis instance a copy of
# another Redis server. Note that the configuration is local to the slave
# so for example it is possible to configure the slave to save the DB with a
# different interval, or to listen to another port, and so on.
#
# slaveof <masterip> <masterport>
```

スレーブ側のRedisに設定を記載します。  
マスター側とDB保存間隔の設定や接続時のポート番号が異なっていたとしても何ら問題はありません。  

##### マスターのパスワード設定
マスター側がパスワード認証を必要している場合、スレーブ側でマスターのパスワードを設定していないとリクエスト要求が通りません。  

```javascript
# If the master is password protected (using the "requirepass" configuration
# directive below) it is possible to tell the slave to authenticate before
# starting the replication synchronization process, otherwise the master will
# refuse the slave request.
#
# masterauth <master-password>
```

##### マスターとの同期が最新でない場合のスレーブ挙動の設定
スレーブがマスターと同期が取れなくなることは長く運用を続けていく上で必ず発生しうるものです。  
その際にスレーブがどのような振る舞いをすべきか設定しておきましょう。  

```javascript
# When a slave loses its connection with the master, or when the replication
# is still in progress, the slave can act in two different ways:
#
# 1) if slave-serve-stale-data is set to 'yes' (the default) the slave will
#    still reply to client requests, possibly with out of date data, or the
#    data set may just be empty if this is the first synchronization.
#
# 2) if slave-serve-stale-data is set to 'no' the slave will reply with
#    an error "SYNC with master in progress" to all the kind of commands
#    but to INFO and SLAVEOF.
#
slave-serve-stale-data yes
```

* `yes`: 最後に同期が取れた際の最新データを返却  
* `no`: 最新の同期が取れていないためエラーを返却  

##### スレーブの書き込み権限の設定
特別な理由がない限り、`yes`で良さそうな設定です。  
スレーブは基本的にマスターに障害が発生した場合に利用する想定なので、直接書き込み権限をつける必要はないでしょう。  

```javascript
# You can configure a slave instance to accept writes or not. Writing against
# a slave instance may be useful to store some ephemeral data (because data
# written on a slave will be easily deleted after resync with the master) but
# may also cause problems if clients are writing to it because of a
# misconfiguration.
#
# Since Redis 2.6 by default slaves are read-only.
#
# Note: read only slaves are not designed to be exposed to untrusted clients
# on the internet. It‘s just a protection layer against misuse of the instance.
# Still a read only slave exports by default all the administrative commands
# such as CONFIG, DEBUG, and so forth. To a limited extent you can improve
# security of read only slaves using 'rename-command' to shadow all the
# administrative / dangerous commands.
slave-read-only yes
```

注意すべきこととしては、`slave-read-only`を`yes`に設定したとしても、`CONFIG`コマンドなどで設定値を変更することができてしまいます。  
もし、それを防止したいのであれば、`rename-command`を利用することで予めコマンド名を第三者からはわからないもに変えておきましょう。  

##### マスターへのヘルスチェック設定
スレーブがマスターの生存を確認するために流し込む`ping`の時間の間隔設定です。

```javascript
# Slaves send PINGs to server in a predefined interval. It‘s possible to change
# this interval with the repl_ping_slave_period option. The default value is 10
# seconds.
#
# repl-ping-slave-period 10
```

##### レプリケーションのタイムアウト設定
マスタースレーブ間で同期を取る際などにリクエスト/レスポンスが発生します。  
その際のタイムアウト設定となります。  

```javascript
# The following option sets the replication timeout for:
#
# 1) Bulk transfer I/O during SYNC, from the point of view of slave.
# 2) Master timeout from the point of view of slaves (data, pings).
# 3) Slave timeout from the point of view of masters (REPLCONF ACK pings).
#
# It is important to make sure that this value is greater than the value
# specified for repl-ping-slave-period otherwise a timeout will be detected
# every time there is low traffic between the master and the slave.
#
# repl-timeout 60
```

注意点としては、`repl-ping-slave-period`よりも大きな設定をしないと毎回`ping`でタイムアウトすることになります。  

##### TCPの遅延に関する設定
TCP通信時の帯域幅の占有可否と遅延の許容可否の優先度を決めます。  

```javascript
# Disable TCP_NODELAY on the slave socket after SYNC?
#
# If you select "yes" Redis will use a smaller number of TCP packets and
# less bandwidth to send data to slaves. But this can add a delay for
# the data to appear on the slave side, up to 40 milliseconds with
# Linux kernels using a default configuration.
#
# If you select "no" the delay for data to appear on the slave side will
# be reduced but more bandwidth will be used for replication.
#
# By default we optimize for low latency, but in very high traffic conditions
# or when the master and slaves are many hops away, turning this to "yes" may
# be a good idea.
repl-disable-tcp-nodelay no
```

* `yes`: TCP通信時のパケット数を小さく抑えることで、帯域幅の占有率を下げることができます。一方で遅延は増加します。  
* `no`: TCP通信時に帯域幅を広く使うことで遅延を減少させます。  

##### スレーブのバックアップログサイズの設定
マスターからスレーブへの接続が切断されてしまっている間に保存しておく差分の最大サイズです。  

```javascript
# Set the replication backlog size. The backlog is a buffer that accumulates
# slave data when slaves are disconnected for some time, so that when a slave
# wants to reconnect again, often a full resync is not needed, but a partial
# resync is enough, just passing the portion of data the slave missed while
# disconnected.
#
# The biggest the replication backlog, the longer the time the slave can be
# disconnected and later be able to perform a partial resynchronization.
#
# The backlog is only allocated once there is at least a slave connected.
#
# repl-backlog-size 1mb
```

##### バックログファイルの維持許容時間の設定
スレーブが切断されている間にバックアップのためのログ保存を行う一方で、そのバックログファイルを解放する設定も可能です。  

```javascript
# After a master has no longer connected slaves for some time, the backlog
# will be freed. The following option configures the amount of seconds that
# need to elapse, starting from the time the last slave disconnected, for
# the backlog buffer to be freed.
#
# A value of 0 means to never release the backlog.
#
# repl-backlog-ttl 3600
```

上記設定でコメントアウトを外せば、1時間はバックアップログファイルを維持します。  
`0`を設定した場合、ファイルの解放は実施しません。  

##### スレーブの優先度設定
マスターが何らかの障害により動作できなくなった場合にスレーブがマスターに昇格することになります。  
もし、スレーブが複数台存在していたら、どれをマスターに昇格させるべきか選ぶ必要があります。  
その際に参考とする値となっています。(実際には `Redis Sentinel` が参考にする値)  

```javascript
# The slave priority is an integer number published by Redis in the INFO output.
# It is used by Redis Sentinel in order to select a slave to promote into a
# master if the master is no longer working correctly.
#
# A slave with a low priority number is considered better for promotion, so
# for instance if there are three slaves with priority 10, 100, 25 Sentinel will
# pick the one with priority 10, that is the lowest.
#
# However a special priority of 0 marks the slave as not able to perform the
# role of master, so a slave with priority of 0 will never be selected by
# Redis Sentinel for promotion.
#
# By default the priority is 100.
slave-priority 100
```

##### マスターへの書き込み権限の設定
Redis2.8系から導入された設定です。  
『マスターに最低 N スレーブが接続している場合にかぎり、書き込み要求を受け付ける設定』です。  

```javascript
# It is possible for a master to stop accepting writes if there are less than
# N slaves connected, having a lag less or equal than M seconds.
#
# The N slaves need to be in "online" state.
#
# The lag in seconds, that must be <= the specified value, is calculated from
# the last ping received from the slave, that is usually sent every second.
#
# This option does not GUARANTEES that N replicas will accept the write, but
# will limit the window of exposure for lost writes in case not enough slaves
# are available, to the specified number of seconds.
#
# For example to require at least 3 slaves with a lag <= 10 seconds use:
#
# min-slaves-to-write 3
# min-slaves-max-lag 10
#
# Setting one or the other to 0 disables the feature.
#
# By default min-slaves-to-write is set to 0 (feature disabled) and
# min-slaves-max-lag is set to 10.
```

* `min-slaves-to-write`: 書き込むために必要な最少接続スレーブ数の設定  
* `min-slaves-max-lag`: 書き込むために必要な最大タイムラグの設定  

#### セキュリティ関連の設定
セキュリティ面で設定が必要な場合に活用します。  

##### 認証パスワードの設定
クライアントからコマンドを叩かれる前にパスワードを要求することでセキュリティを高めます。  
クライアントからRedisに対して直接コマンドを叩くケースがある場合は設定しておきましょう。  

```javascript
# Require clients to issue AUTH <PASSWORD> before processing any other
# commands.  This might be useful in environments in which you do not trust
# others with access to the host running redis-server.
#
# This should stay commented out for backward compatibility and because most
# people do not need auth (e.g. they run their own servers).
#
# Warning: since Redis is pretty fast an outside user can try up to
# 150k passwords per second against a good box. This means that you should
# use a very strong password otherwise it will be very easy to break.
#
# requirepass foobared
```

##### コマンド名の変更
`slave-read-only`で少し触れましたが、`CONFIG`コマンドを直接クライアントから叩かれたりすると困るかもしれません。  
そういったケースが想定される場合は設定しておきましょう。  

```javascript
# Command renaming.
#
# It is possible to change the name of dangerous commands in a shared
# environment. For instance the CONFIG command may be renamed into something
# hard to guess so that it will still be available for internal-use tools
# but not available for general clients.
#
# Example:
#
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
#
# It is also possible to completely kill a command by renaming it into
# an empty string:
#
# rename-command CONFIG ""
#
# Please note that changing the name of commands that are logged into the
# AOF file or transmitted to slaves may cause problems.
```

#### リソース制限に関する設定
サーバはリソースが限られているため、利用ケースに合わせてリソース制限を適切に設定することが必要です。  

##### 最大同時接続クライアント数の設定
デフォルト`10000`ですが、実際にはRedis Serverの予約数である`32`がそこから引かれます。  
もし、設定した同時接続数を超えた場合はエラーを返却します。  

```javascript
# Set the max number of connected clients at the same time. By default
# this limit is set to 10000 clients, however if the Redis server is not
# able to configure the process file limit to allow for the specified limit
# the max number of allowed clients is set to the current file limit
# minus 32 (as Redis reserves a few file descriptors for internal uses).
#
# Once the limit is reached Redis will close all the new connections sending
# an error 'max number of clients reached'.
#
# maxclients 10000
```

##### 利用するメモリ上限の設定
指定した分までメモリを利用します。  
デフォルトはコメントアウトされているため、システム上限までメモリを利用できます。  
上限までメモリを利用した場合の挙動については`maxmemory-policy`で指定します。  

```javascript
# Don‘t use more memory than the specified amount of bytes.
# When the memory limit is reached Redis will try to remove keys
# according to the eviction policy selected (see maxmemory-policy).
#
# If Redis can‘t remove keys according to the policy, or if the policy is
# set to 'noeviction', Redis will start to reply with errors to commands
# that would use more memory, like SET, LPUSH, and so on, and will continue
# to reply to read-only commands like GET.
#
# This option is usually useful when using Redis as an LRU cache, or to set
# a hard memory limit for an instance (using the 'noeviction' policy).
#
# WARNING: If you have slaves attached to an instance with maxmemory on,
# the size of the output buffers needed to feed the slaves are subtracted
# from the used memory count, so that network problems / resyncs will
# not trigger a loop where keys are evicted, and in turn the output
# buffer of slaves is full with DELs of keys evicted triggering the deletion
# of more keys, and so forth until the database is completely emptied.
#
# In short... if you have slaves attached it is suggested that you set a lower
# limit for maxmemory so that there is some free RAM on the system for slave
# output buffers (but this is not needed if the policy is 'noeviction').
#
# maxmemory <bytes>
```

##### 許容メモリ利用上限に達した場合の挙動の設定
幾つか用意されたポリシーの中から挙動を選択します。  

```javascript
# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached. You can select among five behaviors:
#
# volatile-lru -> remove the key with an expire set using an LRU algorithm
# allkeys-lru -> remove any key accordingly to the LRU algorithm
# volatile-random -> remove a random key with an expire set
# allkeys-random -> remove a random key, any key
# volatile-ttl -> remove the key with the nearest expire time (minor TTL)
# noeviction -> don‘t expire at all, just return an error on write operations
#
# Note: with any of the above policies, Redis will return an error on write
#       operations, when there are not suitable keys for eviction.
#
#       At the date of writing this commands are: set setnx setex append
#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
#       getset mset msetnx exec sort
#
# The default is:
#
# maxmemory-policy volatile-lru
```

* `volatile-lru`: LRUアルゴリズムに基づき、期限切れ`key`を削除  
* `allkeys-lru`: LRUアルゴリズムに基づき、期限に関係なく`key`を削除  
* `volatile-random`: 期限切れ`key`をランダム削除  
* `allkeys-random`: 全ての`key`を対象にランダム削除  
* `volatile-ttl`: 期限切れの中でも最も古い`key`から順に削除  
* `noeviction`: `key`に期限切れが存在しないため、上限以上の書き込みに対してはエラーを返却する    

LRUアルゴリズムについては[Least Recently Used - Wikipedia](https://ja.wikipedia.org/wiki/Least_Recently_Used)を読みましょう。  

##### 近似LRUアルゴリズムの精緻設定
先程LRUアルゴリズムを利用する設定があることに触れましたが、Redisではメモリ消費を抑えるために正確なLRUアルゴリズムを採用してはいません。  
代替手段で近似的LRUアルゴリズムを実現しているのですが、その精緻をサンプラーの数を設定することで自由度高く決めることができます。  

```javascript
# LRU and minimal TTL algorithms are not precise algorithms but approximated
# algorithms (in order to save memory), so you can select as well the sample
# size to check. For instance for default Redis will check three keys and
# pick the one that was used less recently, you can change the sample size
# using the following configuration directive.
#
# maxmemory-samples 3
```

#### AOFに関する設定
AOFとは`Append Only File`の略語で追記専用ファイルを利用することによるデータ永続化を目指す設定です。  

##### AOFの利用設定
AOFの利用可否を設定します。もちろん`yes`にすればAOFが利用できます。  

```javascript
# By default Redis asynchronously dumps the dataset on disk. This mode is
# good enough in many applications, but an issue with the Redis process or
# a power outage may result into a few minutes of writes lost (depending on
# the configured save points).
#
# The Append Only File is an alternative persistence mode that provides
# much better durability. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
# Please check http://redis.io/topics/persistence for more information.

appendonly no
```

##### AOFファイル名の設定
ファイル名を設定します。  

```javascript
# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof"
```

##### AOFへの同期タイミングの設定
Redisには3つのモードが存在します。  

* `no`: `fsync`を利用せずOSに全てを任せます。高速です。  
* `always`: 追記専用ログへの書き込み発生する際に必ず`fsync`を実行します。遅いですが安全です。  
* `everysec`: 毎秒ごとに`fsync`を実行します。  

```javascript
# The fsync() call tells the Operating System to actually write data on disk
# instead to wait for more data in the output buffer. Some OS will really flush
# data on disk, some other OS will just try to do it ASAP.
#
# Redis supports three different modes:
#
# no: don‘t fsync, just let the OS flush the data when it wants. Faster.
# always: fsync after every write to the append only log . Slow, Safest.
# everysec: fsync only one time every second. Compromise.
#
# The default is "everysec", as that‘s usually the right compromise between
# speed and data safety. It‘s up to you to understand if you can relax this to
# "no" that will let the operating system flush the output buffer when
# it wants, for better performances (but if you can live with the idea of
# some data loss consider the default persistence mode that‘s snapshotting),
# or on the contrary, use "always" that‘s very slow but a bit safer than
# everysec.
#
# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
#
# If unsure, use "everysec".

# appendfsync always
appendfsync everysec
# appendfsync no
```

デフォルトは`everysec`が設定されています。  

##### ディスクI/O遅延に対する設定
`save`プロセスがディスクに対して大量のI/Oを発生させていた場合、`fsync`呼び出し時にRedisが長時間ブロックする可能性があります。  
バックグラウンドプロセスで`save`が動いている場合は`fsync`を止めるように設定することで、この問題を回避することができます。  
(`no-appendfsync-on-rewrite`は文字通りそういった意味ですね。)  

```javascript
# When the AOF fsync policy is set to always or everysec, and a background
# saving process (a background save or AOF log background rewriting) is
# performing a lot of I/O against the disk, in some Linux configurations
# Redis may block too long on the fsync() call. Note that there is no fix for
# this currently, as even performing fsync in a different thread will block
# our synchronous write(2) call.
#
# In order to mitigate this problem it‘s possible to use the following option
# that will prevent fsync() from being called in the main process while a
# BGSAVE or BGREWRITEAOF is in progress.
#
# This means that while another child is saving, the durability of Redis is
# the same as "appendfsync none". In practical terms, this means that it is
# possible to lose up to 30 seconds of log in the worst scenario (with the
# default Linux settings).
#
# If you have latency problems turn this to "yes". Otherwise leave it as
# "no" that is the safest pick from the point of view of durability.

no-appendfsync-on-rewrite no
```

* `yes`: バックグランドプロセスで`save`が動いていたら`fsync`しない  
* `no`: 上記縛りを設定しない  

##### AOFのログファイル自動書き換え設定
AOFファイルサイズが肥大したときに対処するための基準を設定します。  

```javascript
# Automatic rewrite of the append only file.
# Redis is able to automatically rewrite the log file implicitly calling
# BGREWRITEAOF when the AOF log size grows by the specified percentage.
#
# This is how it works: Redis remembers the size of the AOF file after the
# latest rewrite (if no rewrite has happened since the restart, the size of
# the AOF at startup is used).
#
# This base size is compared to the current size. If the current size is
# bigger than the specified percentage, the rewrite is triggered. Also
# you need to specify a minimal size for the AOF file to be rewritten, this
# is useful to avoid rewriting the AOF file even if the percentage increase
# is reached but it is still pretty small.
#
# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

* `auto-aof-rewrite-percentage`: 書き換えを発生させるファイルサイズの基準  
* `auto-aof-rewrite-min-size`: ここで指定したファイルサイズ以内であれば書き換えを実行しません  

#### LUAスクリプトに関する設定
LUAスクリプトを実行する場合に必要な設定です。  
と言っても最大実行時間しかありません。  

```javascript
# Max execution time of a Lua script in milliseconds.
#
# If the maximum execution time is reached Redis will log that a script is
# still in execution after the maximum allowed time and will start to
# reply to queries with an error.
#
# When a long running script exceed the maximum execution time only the
# SCRIPT KILL and SHUTDOWN NOSAVE commands are available. The first can be
# used to stop a script that did not yet called write commands. The second
# is the only way to shut down the server in the case a write commands was
# already issue by the script but the user don‘t want to wait for the natural
# termination of the script.
#
# Set it to 0 or a negative value for unlimited execution without warnings.
lua-time-limit 5000
```

#### スローログの設定
サービスレベルに耐えられないスロークエリを事前に検出するための設定です。  
サービスレベル上、許容できないスロークエリの閾値を指定することで、その閾値を超えたクエリをログファイルに記録します。  

```javascript
# The Redis Slow Log is a system to log queries that exceeded a specified
# execution time. The execution time does not include the I/O operations
# like talking with the client, sending the reply and so forth,
# but just the time needed to actually execute the command (this is the only
# stage of command execution where the thread is blocked and can not serve
# other requests in the meantime).
#
# You can configure the slow log with two parameters: one tells Redis
# what is the execution time, in microseconds, to exceed in order for the
# command to get logged, and the other parameter is the length of the
# slow log. When a new command is logged the oldest one is removed from the
# queue of logged commands.

# The following time is expressed in microseconds, so 1000000 is equivalent
# to one second. Note that a negative number disables the slow log, while
# a value of zero forces the logging of every command.
slowlog-log-slower-than 10000

# There is no limit to this length. Just be aware that it will consume memory.
# You can reclaim memory used by the slow log with SLOWLOG RESET.
slowlog-max-len 128
```

* `slowlog-log-slower-than`: スロークエリとみなす実行時間の閾値。マイクロ秒です。  
* `slowlog-max-len`: スロークエリログとして記録する容量の閾値。超えると古い順に削除します。  

#### イベント通知設定
Pub/Sub利用時に設定するようです。  
Redisが通知するイベントは幾つかあるのですが、全て1文字で表現されており、`notify-keyspace-events`に指定することで通知イベントを決定します。  

```javascript
# Redis can notify Pub/Sub clients about events happening in the key space.
# This feature is documented at http://redis.io/topics/keyspace-events
#
# For instance if keyspace events notification is enabled, and a client
# performs a DEL operation on key "foo" stored in the Database 0, two
# messages will be published via Pub/Sub:
#
# PUBLISH __keyspace@0__:foo del
# PUBLISH __keyevent@0__:del foo
#
# It is possible to select the events that Redis will notify among a set
# of classes. Every class is identified by a single character:
#
#  K     Keyspace events, published with __keyspace@<db>__ prefix.
#  E     Keyevent events, published with __keyevent@<db>__ prefix.
#  g     Generic commands (non-type specific) like DEL, EXPIRE, RENAME, ...
#  $     String commands
#  l     List commands
#  s     Set commands
#  h     Hash commands
#  z     Sorted set commands
#  x     Expired events (events generated every time a key expires)
#  e     Evicted events (events generated when a key is evicted for maxmemory)
#  A     Alias for g$lshzxe, so that the "AKE" string means all the events.
#
#  The "notify-keyspace-events" takes as argument a string that is composed
#  by zero or multiple characters. The empty string means that notifications
#  are disabled at all.
#
#  Example: to enable list and generic events, from the point of view of the
#           event name, use:
#
#  notify-keyspace-events Elg
#
#  Example 2: to get the stream of the expired keys subscribing to channel
#             name __keyevent@0__:expired use:
#
#  notify-keyspace-events Ex
#
#  By default all notifications are disabled because most users don‘t need
#  this feature and the feature has some overhead. Note that if you don‘t
#  specify at least one of K or E, no events will be delivered.
notify-keyspace-events ""
```

* `AKE`: これを設定することで全ての通知タイプに対応します。  
* `空文字`: これを設定することで全ての通知を無効にします。  

#### 高度な設定
この辺りが意識して触れるようになったらRedis上級者と見なせるのではないでしょうか。  

##### Hash型データのメモリ効率設定
CPUとメモリのトレードオフにはなるのですが、「要素数の上限」と「1要素あたりの上限サイズ」の2つの閾値以内に収まる場合に、非常にメモリ効率の良い方法でエンコードされます。  
これによりメモリが最大10分の1、平均でも5分の1に抑えられます。  

```javascript
# Hashes are encoded using a memory efficient data structure when they have a
# small number of entries, and the biggest entry does not exceed a given
# threshold. These thresholds can be configured using the following directives.
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
```

* `hash-max-ziplist-entries`: 要素数の上限  
* `hash-max-ziplist-value`: 1要素あたりの上限サイズ  

##### List型データのメモリ効率設定
List型データに対する設定です。(各々の意味はHash型と同じです。)  

```javascript
# Similarly to hashes, small lists are also encoded in a special way in order
# to save a lot of space. The special representation is only used when
# you are under the following limits:
list-max-ziplist-entries 512
list-max-ziplist-value 64
```

##### Set型データのメモリ効率設定
Set型データに対する設定です。  
Set型のみ圧縮方式が`ziplist`ではなく`intset`であるためか設定項目数が異なります。  

```javascript
# Sets have a special encoding in just one case: when a set is composed
# of just strings that happens to be integers in radix 10 in the range
# of 64 bit signed integers.
# The following configuration setting sets the limit in the size of the
# set in order to use this special memory saving encoding.
set-max-intset-entries 512
```

##### ZSet型データのメモリ効率設定
ZSet型データに対する設定です。(各々の意味はHash型と同じです。)  

```javascript
# Similarly to hashes and lists, sorted sets are also specially encoded in
# order to save a lot of space. This encoding is only used when the length and
# elements of a sorted set are below the following limits:
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
```

##### 再ハッシュ設定
これは、『100ミリ秒ごとに1ミリ秒のCPU時間を使用して、Redisのメインのハッシュテーブル(トップレベルのキーと値を保持している)の再ハッシュ化を行う』設定です。  
`yes`にすることでメモリが効率的に利用されるようになる一方で最大2ミリ秒間の遅延が発生する可能性があります。  
この2ミリ秒間の遅延を許容できない場合は`no`に設定を変更しましょう。  

```javascript
# Active rehashing uses 1 millisecond every 100 milliseconds of CPU time in
# order to help rehashing the main Redis hash table (the one mapping top-level
# keys to values). The hash table implementation Redis uses (see dict.c)
# performs a lazy rehashing: the more operation you run into a hash table
# that is rehashing, the more rehashing "steps" are performed, so if the
# server is idle the rehashing is never complete and some more memory is used
# by the hash table.
#
# The default is to use this millisecond 10 times every second in order to
# active rehashing the main dictionaries, freeing memory when possible.
#
# If unsure:
# use "activerehashing no" if you have hard latency requirements and it is
# not a good thing in your environment that Redis can reply form time to time
# to queries with 2 milliseconds delay.
#
# use "activerehashing yes" if you don‘t have such hard requirements but
# want to free memory asap when possible.
activerehashing yes
```

2ミリ秒の遅延も許せないサービスってあるんだろうか...と思いつつも。  

##### クライアントの出力バッファ制限の設定

指定した値以上に出力バッファが到達するとクライアントとの接続を強制的に切断します。  
出力バッファが制限値に到達し続けるということは、想定した出力バッファサイズと処理速度を見誤っている可能性があるということです。  


```javascript
# The client output buffer limits can be used to force disconnection of clients
# that are not reading data from the server fast enough for some reason (a
# common reason is that a Pub/Sub client can‘t consume messages as fast as the
# publisher can produce them).
#
# The limit can be set differently for the three different classes of clients:
#
# normal -> normal clients
# slave  -> slave clients and MONITOR clients
# pubsub -> clients subscribed to at least one pubsub channel or pattern
#
# The syntax of every client-output-buffer-limit directive is the following:
#
# client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
#
# A client is immediately disconnected once the hard limit is reached, or if
# the soft limit is reached and remains reached for the specified number of
# seconds (continuously).
# So for instance if the hard limit is 32 megabytes and the soft limit is
# 16 megabytes / 10 seconds, the client will get disconnected immediately
# if the size of the output buffers reach 32 megabytes, but will also get
# disconnected if the client reaches 16 megabytes and continuously overcomes
# the limit for 10 seconds.
#
# By default normal clients are not limited because they don‘t receive data
# without asking (in a push way), but just after a request, so only
# asynchronous clients may create a scenario where data is requested faster
# than it can read.
#
# Instead there is a default limit for pubsub and slave clients, since
# subscribers and slaves receive data in a push fashion.
#
# Both the hard or the soft limit can be disabled by setting them to zero.
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
```

* `client-output-buffer-limit normal 0 0 0`  
  * 通常クライアントに対する設定  
  * hard limit / soft limit を0設定して無効化している  
* `client-output-buffer-limit slave 256mb 64mb 60`  
  * スレーブクライアントに対する設定  
  * hard limitが256mbなので、256MBに到達した時点で即時切断します  
  * soft limitが64mb, soft secondsが60なので、出力バッファが60秒間64MBに到達し続けた場合に切断します  
  * ここはレプリケーション設定にも関わってくるため設定値は考えてつける必要があります  
* `client-output-buffer-limit pubsub 32mb 8mb 60`  
  * Pub/Sub(購読)クライアントに対する設定  
  * hard limitが32mbなので、32MBに到達した時点で即時切断します  
  * soft limitが8mb, soft secondsが60なので、出力バッファが60秒間8MBに到達し続けた場合に切断します  

##### バックグラウンド処理の頻度に関する設定
Redisはバックグランドタスクとして、タイムアウトしたクライアントとのコネクションの切断 / 期限切れキーの削除などを実施しています。  
その頻度を`hz`の値によって決めています。  
基本的に推奨値は`10`なので変更することは少ないと思われます。  
低レイテンシを厳しく求められる環境でのみ`100`を設定しても良いでしょう。  

```javascript
# Redis calls an internal function to perform many background tasks, like
# closing connections of clients in timeout, purging expired keys that are
# never requested, and so forth.
#
# Not all tasks are performed with the same frequency, but Redis checks for
# tasks to perform accordingly to the specified "hz" value.
#
# By default "hz" is set to 10. Raising the value will use more CPU when
# Redis is idle, but at the same time will make Redis more responsive when
# there are many keys expiring at the same time, and timeouts may be
# handled with more precision.
#
# The range is between 1 and 500, however a value over 100 is usually not
# a good idea. Most users should use the default of 10 and raise this up to
# 100 only in environments where very low latency is required.
hz 10
```

##### AOF書き換え時のfsyncの設定

`yes`を設定することで、32MBごとにfsyncを実行するようになります。  
これによりファイルをディスクに書き込む際に、大きな遅延スパイクが発生するのを避けることができます。  

```javascript
# When a child rewrites the AOF file, if the following option is enabled
# the file will be fsync-ed every 32 MB of data generated. This is useful
# in order to commit the file to the disk more incrementally and avoid
# big latency spikes.
aof-rewrite-incremental-fsync yes
```

### まとめ
細かな設定の棚卸しをしてみたものの、かなりしんどかったです...  
運用してみないと想像がつかない設定も多く、この辺りは本当に実務でしか経験を養えないのだと痛感しました。  
しかし、各々の意味を理解しておくことは決して無駄ではないと思うので、勉強になったかなと思います。  
と言ったところで本日はここまで。  

### 参考

* [Redis Documentation レプリケーション](http://redis-documentasion-japanese.readthedocs.io/ja/latest/topics/replication.html)  
* [Redis Documentation LRUキャッシュ](http://redis-documentasion-japanese.readthedocs.io/ja/latest/topics/lru-cache.html)  
* [Redis Documentation キー・スペース通知](http://redis-documentasion-japanese.readthedocs.io/ja/latest/topics/notifications.html)  
* [Redis Documentation メモリ最適化](http://redis-documentasion-japanese.readthedocs.io/ja/latest/topics/memory-optimization.html)  
* [真剣にRedisを使ってみようという気持ちになったのでRedisについて知っていることを書く](http://t-cyrill.hatenablog.jp/entry/2016/12/11/224604)  
* [Redis のメモリが足りなくなった時にどうやってチューニングしたか](http://qiita.com/eccyan/items/e8cc56948a00d6aad0aa)  
* [Redisのメモリ使用量削減](http://mocobeta-backup.tumblr.com/post/84793291682/redis-set)  
* [EC2など 高負荷クラウド環境における Redis のチューニングについて](http://tech.guitarrapc.com/entry/2013/08/02/210858)  
* [redis 2.0.3 documentation](http://redis.shibu.jp/admin/config.html)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
