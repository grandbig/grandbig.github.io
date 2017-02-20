---
layout: post
title: "DockerにWebサーバーを立てよう！"
date: 2017-02-19 15:05
comments: true
categories: docker apache
---

### はじめに
さて、今回は前回([Docker for Macをインストールしよう！](http://grandbig.github.io/blog/2017/02/13/docker-for-mac-1/))構築したDocker内のUbuntuサーバの中で遊んでみます。  
まずは、基本となるWebサーバを立てるところから始めたいと思います。  

今回は最も有名であろう(Nginxも人気ですが...) **Apache** を利用したWebサーバの構築をしてみます。  
限りなく常識に近いと思うものの、一応概要だけ紹介すると、Apacheとは下記の通りです。  

* 正式名所は「 **Apache HTTP Server** 」  
* Webサーバソフトウェア  
* Apache Licenseで無償で提供  
* 2.2系と2.4系があるがモダンなのは **2.4系**  

というところで早速見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### Host Macからアクセスするためのポートフォワード設定
これからApacheをインストールするわけですが、その前にDockerコンテナにポートフォワード設定をしてみましょう。  
Docker for Macなら簡単にできます。  

１．該当コンテナの設定を見ましょう  

![設定を見る](/images/docker_port_forward_1.png)  

２．Port設定を見ましょう  

![Port設定を見る](/images/docker_port_forward_2.png)  

３．ポートフォワードの設定を追加する  

![ポートフォワードの設定を追加](/images/docker_port_forward_3.png)  

上記では、`http://localhost:15600/`でMacからアクセスできるようにしています。  
`localhost`でなく、具体的にIPアドレスを指定しても良いようですが、今回は特に必要ないのでこれでいきます。  

### Apacheのインストール
まずは当然のことながらインストール作業から始めます。  
DockerのUbuntuコンテナを起動した後にログインして、下記コマンドを叩きましょう。  

ログインはDockerアプリに含まれている **Kitematic** からもできますし、コマンドでもできます。  

```javascript
// docker コンテナにログイン
$ docker exec -it <<コンテナ名>> bash

// apt-getのアップデート
$ apt-get update
// Apacheのインストール
$ apt-get install apache2
```

なぜ初めに`apt-get update`をやっているかと言うと、これをやらずに突き進もうとした際に下記エラーが発生したためです。  

```javascript
# apt-get install apache2
Reading package lists... Done
Building dependency tree       
Reading state information... Done
E: Unable to locate package apache2
```

きちんとApacheのインストールが完了したか確認するためにバージョンを見てみましょう。  

```javascript
// バージョン確認方法(1)
$ apachectl -v
Server version: Apache/2.4.18 (Ubuntu)
Server built:   2016-07-14T12:32:26

// バージョン確認方法(2)
$ apache2 -v
Server version: Apache/2.4.18 (Ubuntu)
Server built:   2016-07-14T12:32:26
```

### Apacheの起動確認
インストール後はデフォルトで起動していると思いますが、念のため確認しておきましょう。  

```javascript
$ /etc/init.d/apache2 status
 * apache2 is running
```

因みに、初期状態では`start`や`restart`コマンドを実行したときに下記警告が表示されます。  

```javascript
$ /etc/init.d/apache2 restart
 * Restarting Apache httpd web server apache2                                                                                                                                                           AH00558: apache2: Could not reliably determine the server’s fully qualified domain name, using XX.XX.XX.XX. Set the 'ServerName' directive globally to suppress this message
```

これを回避するためには設定ファイルに`ServerName`を定義する必要があります。  
(警告は出るものの、起動には何ら問題はありません。)  

下記のように設定ファイルを開きます。  

```javascript
// 設定ファイルを開く
$ vi /etc/apache2/apache2.conf
```

続いて、`ServerName`を適当な場所に定義してやりましょう。  

```javascript
// /etc/apache2/apache2.conf
// 省略
# Include list of ports to listen on
Include ports.conf

ServerName xxx.com
// 省略
```

これにより、エラーなく起動や再起動できることを確認できるはずです。  

```javascript
$ /etc/init.d/apache2 restart
 * Restarting Apache httpd web server apache2  
```

これでApacheが起動していることが確認できたので、GETリクエストを投げてアクセスしてみましょう。  
Ubuntu用のApacheデフォルトページが返却されるはずです。  

```javascript
$ curl http://127.0.0.1/

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
  ...
</html>
```

### Webページにアクセス
冒頭でポートフォワード設定をしたので、Apacheが起動しているのであれば、Macからページを見れるはずです！  
早速見てみましょう。  
設定した`http://localhost:15600/`でアクセスすると下記のようにページが表示されます。  

![DockerコンテナのWEBページ](/images/docker_apache_1.png)  

### 便利コマンド利用するためにモジュールをインストール
Webサーバを構築する上で、あまりにもまっさらなUbuntuだったため幾つか必要なものをインストールする必要がありました。  
その紹介もおまけ程度に書いておきます。  

#### lessのインストール
毎回`cat`コマンドで中身を見る必要性がないこともあると思います。  
流石に`less`くらいはインストールしておきます。  

```javascript
// lessのインストール
$ apt-get install less

// lessのバージョン確認
$ less --help
```

#### lessコマンドで色をつける
ついでに`less`コマンドで色を付けて見やすくする方法を書いておきます。  

```javascript
// source-highlightのインストール
$ apt-get install source-highlight
```

Dockerコンテナ内の環境変数は`.dockerenv`ファイル内に書きます。  

```javascript
// .dockerenvに下記を追加
#Source-hilight with less
export LESSOPEN="| /usr/share/source-highlight/src-hilite-lesspipe.sh %s"
export LESS='-R'
```

結果を反映させるために、下記コマンドを叩きましょう。  

```javascript
$ source ~/.dockerenv
```

#### vi/vimのインストール
ファイルを作成する際に必ず必要になります。  

```javascript
// vimのインストール
$ apt-get install vim

// vimのバージョン確認
$ vim -v
// viのバージョン確認
$ vi -v
```

#### curlのインストール
`curl`は試しにHTTPリクエストを投げてみたいことがあると思ったのでインストールしてみました。  

```javascript
// curlのインストール
$ apt-get install curl

// curlのバージョン確認
$ curl -V
curl 7.47.0 (x86_64-pc-linux-gnu) libcurl/7.47.0 GnuTLS/3.4.10 zlib/1.2.8 libidn/1.32 librtmp/2.3
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtmp rtsp smb smbs smtp smtps telnet tftp
Features: AsynchDNS IDN IPv6 Largefile GSS-API Kerberos SPNEGO NTLM NTLM_WB SSL libz TLS-SRP UnixSockets
```

### まとめ
さて一先ず本記事の目的は達成できました。  
が、Apacheのあまりにも簡単な基礎部分しか見ていません。  
引き続き、最低限の設定を見ていきたいと思っています。  

と言ったところで本日はここまで。  

### 参考
以下、ページを参考にさせて頂きました。  

* [Ubuntuでlessを使って構文をカラー表示する方法](http://www.nemotos.net/?p=1100)  
* [docker-machineコマンド](http://qiita.com/maemori/items/e7318b088b9e4bf22310)  
* [Kitematic user guide](https://docs.docker.com/kitematic/userguide/)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
