---
layout: post
title: "Dockerコンテナ内のUbuntuでMariaDBを使ってみよう！"
date: 2017-02-26 22:38
comments: true
categories: docker MariaDB
---

### はじめに
さて、Redisに引き続き[MariaDB](https://mariadb.org/)をDockerコンテナ内のUbuntuにインストールしてみたいと思います。  
今回扱う環境は以下です。  

* Ubuntu: 16.04.2 LTS  
* MariaDB: Ver 15.1 Distrib 10.0.29-MariaDB, for debian-linux-gnu (x86_64) using readline 5.2  

では早速インストール方法から見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### MariaDBのインストール
これまで通りで非常に簡単です。  

```javascript
// 念のためapt-getをupdate && upgrade
$ apt-get update
$ apt-get upgrade

// MariaDBのインストール
$ apt-get install mariadb-server

// MariaDBがインストールされたことの確認
// /etc配下にmysqlフォルダがある
$ cd /etc
$ cd ls
```

### MariaDBの初期設定
インストール後の初期設定手順を書きます。  

```javascript
$ mysql_secure_installation
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we‘ll need the current
password for the root user.  If you‘ve just installed MariaDB, and
you haven‘t set the root password yet, the password will be blank,
so you should just press enter here.

// rootユーザのパスワードを設定しましょう！
Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

You already have a root password set, so you can safely answer 'n'.

// rootユーザのパスワードを変更しますか？(既に設定したので "n" を入力)
Change the root password? [Y/n] n
 ... skipping.

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

// 匿名ユーザを削除しますか？
Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

// リモートでrootによるログインを許可しますか？
Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

// test用のデータベースを削除しますか？
Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

// テーブルへの特権情報をリロードしますか？(上記までの変更を即時反映させるため "y" を選択)
Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you‘ve completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

### MariaDBにユーザを追加
流石にrootユーザだけだと何なので、メインで利用するユーザを作成しましょう。  

```javascript
// MariaDBにログイン
$ mysql -u root -p
// ユーザを作成
MariaDB [(none)]> create user 'takahiro'@'localhost' identified by 'password';
// ユーザの一覧を確認
MariaDB [(none)]> select Host, User, Password from mysql.user;
+-----------+----------+-------------------------------------------+
| Host      | User     | Password                                  |
+-----------+----------+-------------------------------------------+
| localhost | root     |                                           |
| localhost | takahiro | ***************************************** |
+-----------+----------+-------------------------------------------+
```

### MariaDBの設定ファイル
さてさて、続いてMariaDBのデフォルトの設定ファイルを見ていきます。  

設定ファイルがたくさんあることで混乱しそうになりますが、1つ1つはあまり何も書いていません。  
構造を見てみると...  

```javascript
mysql
├── conf.d
│    ├── mysql.cnf
│    └── mysqldump.cnf
├── debian-start
├── debian.cnf
├── mariadb.cnf
├── mariadb.conf.d
│    ├── 50-client.cnf
│    ├── 50-mysql-clients.cnf
│    ├── 50-mysqld_safe.cnf
│    └── 50-server.cnf
├── my.cnf
└── my.cnf.fallback
```

となっています。  
それぞれの大まかな意図は`mariadb.cnf`に書かれています。  

##### mariadb.cnf
このファイルには下記のように書かれています。  

```javascript
# The MariaDB configuration file
#
# The MariaDB/MySQL tools read configuration files in the following order:
# 1. "/etc/mysql/mariadb.cnf" (this file) to set global defaults,
# 2. "/etc/mysql/conf.d/*.cnf" to set global options.
# 3. "/etc/mysql/mariadb.conf.d/*.cnf" to set MariaDB-only options.
# 4. "~/.my.cnf" to set user-specific options.
#
# If the same option is defined multiple times, the last one will apply.
#
# One can use all long options that the program supports.
# Run program with --help to get a list of available options and with
# --print-defaults to see which it would actually understand and use.

#
# This group is read both both by the client and the server
# use it for options that affect everything
#
[client-server]

# Import all .cnf files from configuration directory
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mariadb.conf.d/
```

ここに書かれている通り、  

1. `/etc/mysql/mariadb.cnf`: グローバル設定を書く  
2. `/etc/mysql/conf.d/*.cnf`: グローバルオプション設定を書く
3. `/etc/mysql/mariadb.conf.d/*.cnf`: MariaDBにのみ有効なオプション設定を書く  
4. `~/.my.cnf`: ユーザ固有のオプション設定を書く  

というフォルダ構成になっているようです。  

また、`[client-server]`を設定することでMariaDBクライアントとMariaDBサーバの両方から読み込まれる設定ファイルとして定義しています。  
筆者が少し引っかかったのは`!includedir`構文です。  
これは指定したファイルのインクルードを実行する構文です。  
(` ! `がついているのでインクルードしないのかと思いきや、これでインクルートなんですね...)  

##### conf.d/mysql.cnf

下記しか書かれていません...  
MySQLクライアントに有効な設定を書くことを想定している書き方に見えますね。  

```javascript
[mysql]
```

##### conf.d/mysqldump.cnf

名前からして`mysqldump`に関する設定ですね。  

```javascript
[mysqldump]
quick
quote-names
max_allowed_packet      = 16M
```

* `quick`: テーブルを1行ずつDumpする設定  
* `quote-names`: 識別子を逆引用符文字で囲む  
* `max_allowed_packet`: サーバーとの間で送受信するパケットの最大長  

##### mariadb.conf.d/50-client.cnf

```javascript
#
# This group is read by the client library
# Use it for options that affect all clients, but not the server
#

[client]
# Default is Latin1, if you need UTF-8 set this (also in server section)
default-character-set = utf8mb4

# This group is *never* read by mysql client library, though this
# /etc/mysql/mariadb.cnf.d/client.cnf file is not read by Oracle MySQL
# client anyway.
# If you use the same .cnf file for MySQL and MariaDB,
# use it for MariaDB-only client options
[client-mariadb]
```

MySQL / MariaDBクライアント側のデフォルトの文字コードが`utf8mb4`と設定されています。  
また、MariaDBクライアントのみ設定を追加したい場合は`[client-mariadb]`より下の行にオプション設定を書きましょう。  

##### mariadb.conf.d/50-mysql-clients.cnf

```javascript
#
# These groups are read by MariaDB command-line tools
# Use it for options that affect only one utility
#

[mysql]
# Default is Latin1, if you need UTF-8 set this (also in server section)
default-character-set = utf8mb4

[mysql_upgrade]

[mysqladmin]

[mysqlbinlog]

[mysqlcheck]

[mysqldump]

[mysqlimport]

[mysqlshow]

[mysqlslap]
```

MySQLクライアント側のデフォルト文字コードは`utf8mb4`で設定されています。  
他にもMySQL関連の設定を追加できるようですが、デフォルトでは何も書かれていませんでした。  
一応、それぞれについて簡単に説明しておきます。  

* `mysqladmin`  
  * MySQLサーバの管理を行うクライアント  
  * サーバーの構成や現在のステータスの確認、データベースの作成および削除、およびその他の用途に使用
* `mysqlbinlog`  
  * MySQLのバイナリログの内容を確認  
* `mysqlcheck`  
  * テーブルの保守 (テーブルの検査、修復、最適化、分析) を実行  
* `mysqldump`    
  * MySQLデータのバックアップを実行  
* `mysqlimport`  
  * LOAD DATA INFILE SQL ステートメントにコマンド行インタフェースを提供  
  * ファイルをテーブルにインポート  
* `mysqlshow`  
  * データベース / テーブル / テーブルのカラム / インデックスの存在迅速に確認するために使用  
* `mysqlslap`  
  * MySQL サーバーのクライアント負荷をエミュレートし、各段階のタイミングをレポートする  

##### mariadb.conf.d/50-mysqld_safe.cnf

エラー発生時の再起動やランタイム情報のエラーログファイルへのロギングなど、いくつかの安全機能を追加するために設定します。  

```javascript
[mysqld_safe]
# This will be passed to all mysql clients
# It has been reported that passwords should be enclosed with ticks/quotes
# escpecially if they contain "#" chars...
# Remember to edit /etc/mysql/debian.cnf when changing the socket location.
socket          = /var/run/mysqld/mysqld.sock
nice            = 0
skip_log_error
syslog
```

* `socket`: Unixドメインソケット接続を待機するソケットファイル  
* `nice`: サーバのスケジュール設定の優先順位を設定するために利用します  
* `skip_log_error`: エラーログの出力をスキップする設定  
* `syslog`: エラーメッセージをsyslogに書き込むための設定  


##### mariadb.conf.d/50-server.cnf

文章の長さからしてMariaDB設定のベースとなります。  
分割して見ていきます。  
まずは基本設定です。  

```javascript
#
# These groups are read by MariaDB server.
# Use it for options that only the server (but not clients) should see
#
# See the examples of server my.cnf files in /usr/share/mysql/
#

# this is read by the standalone daemon and embedded servers
[server]

# this is only for the mysqld standalone daemon
[mysqld]

#
# * Basic Settings
#
user            = mysql
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
port            = 3306
basedir         = /usr
datadir         = /var/lib/mysql
tmpdir          = /tmp
lc-messages-dir = /usr/share/mysql
skip-external-locking

# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address            = 127.0.0.1
```

* `user`: MariaDBに関連する操作を実行するためのユーザを設定します  
* `pid-file`: PIDファイルの指定先です  
* `socket`: Unixドメインソケット接続を待機するソケットファイル  
* `port`: 接続するためのポート番号です  
* `basedir`: MariaDBのインストールディレクトリ  
* `datadir`: MariaDBのデータベースに関するファイルが格納されるディレクトリ  
* `tmpdir`: ファイルシステムのディレクトリ  
* `lc-messages-dir`: エラーメッセージの配置ディレクトリ  
* `skip-external-locking`: 外部ロックを無効にする設定  
* `bind-address`: MariaDBへの接続を許可するIPアドレスの設定  

続いて、チューニング周りです。  

```javascript
#
# * Fine Tuning
#
key_buffer_size         = 16M
max_allowed_packet      = 16M
thread_stack            = 192K
thread_cache_size       = 8
# This replaces the startup script and checks MyISAM tables if needed
# the first time they are touched
myisam-recover         = BACKUP
#max_connections        = 100
#table_cache            = 64
#thread_concurrency     = 10
```

* `key_buffer_size`: インデックスをメモリ上で確保するために使う物理メモリの許容量  
* `max_allowed_packet`: クライアントからMySQLサーバに転送可能な最大パケット数  
* `thread_stack`: スレッドに割り当てるスタック領域のサイズ  
* `thread_cache_size`: キャッシュに保持するスレッドの最大数  
* `myisam-recover `:MyISAMテーブルがクラッシュしたときに自動でリカバリーする設定  
* `max_connections`: 許容同時接続数  
* `table_cache`: メモリ上にキャッシュとして保持しておくテーブルの最大数  
* `thread_concurrency`: 同時実行スレッド数  

クエリキャッシュの設定です。  

```javascript
#
# * Query Cache Configuration
#
query_cache_limit       = 1M
query_cache_size        = 16M
```

* `query_cache_limit`: キャッシュするクエリ結果の最大値  
* `query_cache_size`: クエリキャッシュのサイズ設定  

ロギングとレプリケーションに関する設定です。  

```javascript
#
# * Logging and Replication
#
# Both location gets rotated by the cronjob.
# Be aware that this log type is a performance killer.
# As of 5.1 you can enable the log at runtime!
#general_log_file        = /var/log/mysql/mysql.log
#general_log             = 1
#
# Error log - should be very few entries.
#
log_error = /var/log/mysql/error.log
#
# Enable the slow query log to see queries with especially long duration
#slow_query_log_file    = /var/log/mysql/mariadb-slow.log
#long_query_time = 10
#log_slow_rate_limit    = 1000
#log_slow_verbosity     = query_plan
#log-queries-not-using-indexes
#
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replication slave, see README.Debian about
#       other settings you may need to change.
#server-id              = 1
#log_bin                        = /var/log/mysql/mysql-bin.log
expire_logs_days        = 10
max_binlog_size   = 100M
#binlog_do_db           = include_database_name
#binlog_ignore_db       = include_database_name
```

* `general_log_file`: 一般クエリーログの保存ファイルを指定  
* `general_log`: 1を指定で一般クエリーログの取得が有効になる  
* `log_error`: エラーログの保存ファイルを指定  
* `slow_query_log_file`: スロークエリーログの保存ファイルを指定  
* `long_query_time`: スロークエリーと判断するための時間(秒)  
* `log_slow_rate_limit`: スロークエリーを判断するタイミングを設定 (1なら毎クエリごとに調査)  
* `log_slow_verbosity`: どの程度の詳細レベルでログを出すか決める  
* `log-queries-not-using-indexes`: Indexを使わない, 使っても全件スキャンになっているクエリはスロークエリとして出力する  
* `server-id`: 一意なサーバを識別するために設定(レプリケーション組む時に使える)  
* `log_bin`: バイナリログファイルを保存するファイルの指定  
* `expire_logs_day`: バイナリログの保存期限日  
* `max_binlog_size`: バイナリログろログローテーションし始める基準の値  
* `binlog_do_db`: バイナリログに書き込むデータベースを指定  
* `binlog_ignore_db`: バイナリログに書き込まないデータベースを指定  

以下はInnoDB特有の設定のようです。  

```javascript
#
# * InnoDB
#
# InnoDB is enabled by default with a 10MB datafile in /var/lib/mysql/.
# Read the manual for more InnoDB related options. There are many!

#
# * Security Features
#
# Read the manual, too, if you want chroot!
# chroot = /var/lib/mysql/
#
# For generating SSL certificates I recommend the OpenSSL GUI "tinyca".
#
# ssl-ca=/etc/mysql/cacert.pem
# ssl-cert=/etc/mysql/server-cert.pem
# ssl-key=/etc/mysql/server-key.pem
```

* `ssl-ca`: SSL接続のためのCA証明書の指定  
* `ssl-cert`: SSL接続のためのサーバ証明書の指定  
* `ssl-key`: SSL接続のための秘密鍵の指定  

エンコードの設定です。  

```javascript
#
# * Character sets
#
# MySQL/MariaDB default is Latin1, but in Debian we rather default to the full
# utf8 4-byte character set. See also client.cnf
#
character-set-server  = utf8mb4
collation-server      = utf8mb4_general_ci
```

* `character-set-server`: MySQLサーバの文字コード設定  
* `collation-server`: MySQLサーバのソート順設定  

その他にオプションを設定する場合は下記に続けて書くように用意されています。  

```javascript
#
# * Unix socket authentication plugin is built-in since 10.0.22-6
#
# Needed so the root database user can authenticate without a password but
# only when running as the unix root user.
#
# Also available for other users if required.
# See https://mariadb.com/kb/en/unix_socket-authentication-plugin/

# this is only for embedded server
[embedded]

# This group is only read by MariaDB servers, not by MySQL.
# If you use the same .cnf file for MySQL and MariaDB,
# you can put MariaDB-only options here
[mariadb]

# This group is only read by MariaDB-10.0 servers.
# If you use the same .cnf file for MariaDB of different versions,
# use this group for options that older servers don‘t understand
[mariadb-10.0]
```

### まとめ
今回はインストールと初期設定、デフォルトで生成される設定ファイルの意味について見ていきました。  
今後は理解を深めるためにMariaDBならではのアーキテクチャを組んで試していければと思います。  
と言ったところで本日はここまで。  

### 参考

* [4.5.4 mysqldump — データベースバックアッププログラム](https://dev.mysql.com/doc/refman/5.6/ja/mysqldump.html)  
* [4.2.6 オプションファイルの使用](https://dev.mysql.com/doc/refman/5.6/ja/option-files.html)  
* [mysqld_safe Options](https://mariadb.com/kb/en/mariadb/mysqld_safe/)  
* [MariaDB System Variable](https://mariadb.com/kb/en/mariadb/server-system-variables/#log_queries_not_using_indexes)

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
