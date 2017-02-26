---
layout: post
title: "Dockerコンテナ内のUbuntuでApache設定を試してみよう！"
date: 2017-02-19 23:45
comments: true
categories: docker apache
---

### はじめに
前回の[DockerにWebサーバーを立てよう！](http://grandbig.github.io/blog/2017/02/19/docker-for-mac-2/)に引き続きDocker内Apacheで遊んでみます。  
前回はUbuntuへのApacheインストールから起動まで見てきました。  
実際の現場でApacheを利用する際は様々な設定を施す必要があります。  
本記事ではその一端を少しでも学ぼうということで書いていきます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### DocumentRootの設定
Apacheでは設定ファイルで静的ファイルのルートパスを設定することができます。  
デフォルトでは`/var/www/html`になっています。  
これは、`/etc/apache2/sites-enabled/000-default.conf`で次のように定義されています。  

```javascript
<VirtualHost *:80>
  # The ServerName directive sets the request scheme, hostname and port that
  # the server uses to identify itself. This is used when creating
  # redirection URLs. In the context of virtual hosts, the ServerName
  # specifies what hostname must appear in the request’s Host: header to
  # match this virtual host. For the default virtual host (this file) this
  # value is not decisive as it is used as a last resort host regardless.
  # However, you must set it for any further virtual host explicitly.
  #ServerName www.example.com

  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/html

  # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
  # error, crit, alert, emerg.
  # It is also possible to configure the loglevel for particular
  # modules, e.g.
  #LogLevel info ssl:warn

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined

  # For most configuration files from conf-available/, which are
  # enabled or disabled at a global level, it is possible to
  # include a line for only one particular virtual host. For example the
  # following line enables the CGI configuration for this host only
  # after it has been globally disabled with "a2disconf".
  #Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

必要があれば書き換えて使いましょう。  

### リバースプロキシ設定
リバースプロキシとは、  

* クライアントからのアクセスをキャッチして、特定のサーバへ送るプロキシサーバ  
* セキュリティや負荷分散などのために利用される  

ものです。  
詳しくは[リバースプロキシ - wiki](https://ja.wikipedia.org/wiki/%E3%83%AA%E3%83%90%E3%83%BC%E3%82%B9%E3%83%97%E3%83%AD%E3%82%AD%E3%82%B7)を見てください。  

設定はいたって簡単です。  
まずは、モジュールの有効化をするために下記コマンドを打ちましょう。  

```javascript
// プロキシモジュールの有効化
$ a2enmod proxy proxy_http
Enabling module proxy.
Considering dependency proxy for proxy_http:
Module proxy already enabled
Enabling module proxy_http.
To activate the new configuration, you need to run:
  service apache2 restart

// Apacheを再起動
$ /etc/init.d/apache2 restart
```

続いて、設定ファイルを更新します。  

```javascript
<IfModule mod_proxy.c>

  # If you want to use apache2 as a forward proxy, uncomment the
  # 'ProxyRequests On' line and the <Proxy *> block below.
  # WARNING: Be careful to restrict access inside the <Proxy *> block.
  # Open proxy servers are dangerous both to your network and to the
  # Internet at large.
  #
  # If you only want to use apache2 as a reverse proxy/gateway in
  # front of some web application server, you DON’T need
  ProxyRequests Off

  <Proxy *>
  #   AddDefaultCharset off
  #   Require all denied
  #   #Require local
    Require all granted
  </Proxy>
  ProxyPass /hoge/ http://localhost:80/hoge.html
  ProxyPassReverse /hoge/ http://localhost:80/hoge.html

  # Enable/disable the handling of HTTP/1.1 "Via:" headers.
  # ("Full" adds the server version; "Block" removes all outgoing Via: headers)
  # Set to one of: Off | On | Full | Block
  #ProxyVia Off
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

上記の設定について説明します。  

* `ProxyRequests Off`: フォワードプロキシをOFFにする設定です  
* `<Proxy *> 〜 </Proxy>`: アクセスパスに対するアクセス権限範囲を設定します  
* `ProxyPass 元のアクセス要求パス 転送パス`: クライアントからのアクセス要求を転送します  
* `ProxyPassReverse 元のアクセス要求パス 転送パス`: Apache に HTTP リダイレクト応答の Location, Content-Location, URI ヘッダの調整を担います  

筆者自身、フォワードプロキシという単語を意識して使ったことがなかったのですが、ごくごく普通に使われているものなんですよね...  
Apache公式ページの下記記述を見るとよくわかります。  

> ファイアウォールによって 制限されている内部のクライアントにインターネットへのアクセスを 提供するもの

どうも`ProxyPassReverse`は理解するのが厄介だったようで多くの方が記事にしてくれていました。  

* [mod_proxy再入門 – ProxyPassとProxyPassReverse](http://dev.classmethod.jp/server-side/server/introduction_mod_proxy/)  
* [やっとわかった、リバースプロキシの設定の意味](http://d.hatena.ne.jp/a666666/20090211/1234348004)  

上記設定をしたので下記にアクセスしてみます。  
`http://localhost:15600/hoge/`  
結果は下記の通りです。  

![リバースプロキシした画面を表示](/images/docker_reverse_proxy.png)  

このために用意した`hoge.html`が表示されました。  

### RewriteRuleの設定
`Rewrite`はクライアントからのリクエストを内部変換してリダイレクトするような機能です。  
今回は些か簡単な例を扱ったためリバースプロキシとの違いがわかりにくかったかもしれません。  
あくまでも`Rewrite`はパスの読み替えであるため、物理的に別のサーバに接続を促すようなリバースプロキシならではの設定はできません。  

まずはモジュールを有効化させます。  

```javascript
$ a2enmod rewrite
Enabling module rewrite.
To activate the new configuration, you need to run:
  service apache2 restart

// Apacheを再起動
$ /etc/init.d/apache2 restart
```

続いて、静的ファイルパスへのアクセス制限を変更します。  

```javascript
// /etc/apache2/apache2.conf
<Directory /var/www/>
  Options Indexes FollowSymLinks
  AllowOverride All
  Require all granted
</Directory>
```

デフォルトでは上記が`AllowOverride None`だったのを`AllowOverride All`に変更しています。  
さらに、`.htaccess`ファイルを作成して書き換えルールを記載します。  
`.htaccess`を利用する設定は`apache2.conf`にデフォルトで記載されています。  

```javascript
// /etc/apache2/apache2.conf
# AccessFileName: The name of the file to look for in each directory
# for additional configuration directives.  See also the AllowOverride
# directive.
#
AccessFileName .htaccess
```

そして実際の`RewriteRule`の記載です。  

```javascript
// /var/www/html/.htaccess
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteRule ^fuga/(.*)$ /bar/$1
</IfModule>
```

サンプル用に、`/var/www/html`配下に`bar`フォルダを作成し、その配下に`index.html`を作成しました。  
上記設定をしたので下記にアクセスしてみます。  
`http://localhost:15600/fuga/`  
結果は下記の通りです。  

![RewriteRule適用](/images/docker_rewrite.png)  

### ErrorDocumentの設定
エラー発生時にカスタムエラーページを表示したいときなどに利用します。  
具体的には、  

```javascript
// /etc/apache2/site-enabled/000-default.conf
<VirtualHost *:80>
  ...
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined

  // 以下、ErrorDocumentの設定
  ErrorDocument 404 error_404.html
  ErrorDocument 500 error_500.html
  ...
</VirtualHost>
```

のように設定が可能です。  
Ubuntuの場合、`/etc/apache2/conf-enabled/localized-error-pages.conf`に`ErrorDocument`の構文が全てコメントアウトの状態で記載があるため、本格的にやるならきちんとファイルを分けた方が良さそうです。

### まとめ
さて今回はApacheの設定を少し見ることができました。  
WEBサーバとはどんなものなのか、わかったようでわかっていなかったことが見えてきた気がします。  
引き続き勉強を続けていきたいものです。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
