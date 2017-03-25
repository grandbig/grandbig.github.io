---
layout: post
title: "Kitematicでエラーが発生してDockerコンテナが作成できない！！"
date: 2017-03-25 20:01
comments: true
categories: docker
---

### はじめに
前回からだいぶ期間が空いてしまいましたが、久々に書きます！  
と言っても今回はDocker For Macアプリに備え付けられている **Kitematic** に関してのメモです。  

### KitematicでDockerイメージをダウンロードできない場合の対応
KitematicはGUIで簡単にDockerコンテナを作成できたり、**ポートフォワード** や **Privileged mode** などの設定も実に簡単です。  
その利便性が頼りがちになってしまうものの、Docker For Macアプリも完璧ではないため、まだまだうまくいかないことが時たまあります。  
その1つが『Kitematicを通して、Dockerイメージがダウンロードできない』というものです。   

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

もし、下図のことが発生したら、次の操作を試してみましょう。 

![Dockerイメージのダウンロードに失敗](/images/docker_pull_failed.png)  

```javascript
$ docker pull <Dockerイメージ名>
```

これで該当するDockerイメージをダウンロードすることができます。  

#### Dockerコンテナでsystemctlコマンドが叩けない
上記対応を進める中でぶつかった話なので書いておきます。  
CentOS7.3のDockerイメージをダウンロードして、Dockerコンテナを起動して、Apacheをダウンロード&起動しようとしたところ、表題の問題にぶつかりました。  

```javascript
$ systemctl status httpd.service
Failed to get D-Bus connection: Operation not permitted
```

これについて調べてみると、`Privileged mode`をONにした状態でDockerコンテナを初期化&起動する必要があることがわかりました。  
つまり、コマンドで言うところの  

```javascript
$ docker run --privileged -d --name httpd centos:latest /sbin/init
```

という感じです。  
因みに上記コマンドは、`docker pull`の役割も同時に果たしています。  
これまた `Kitematic` の設定でチェックをつけて保存し直してもうまくいかず...  

#### ポートフォワードつきDockerコンテナの初期化
さて、せっかくなのでポートフォワードの設定もコマンドベースで書いていきます。  

```javascript
$ docker run -p 8888:80 --name httpd centos:latest /sbin/init
```

これで、 `http://localhost:8888`でアクセスしたときにDockerコンテナの`80`ポートにアクセスすることができるようになります。  
`Kitematic`でポートフォワードの設定はできるのですが、`Privileged mode`も必要なら下記コマンドのように初めに同時に設定してしまえば良いでしょう。  

```javascript
$ docker run --privileged -d -p 8888:80 --name httpd centos:latest /sbin/init
```

### まとめ
今回は`Docker`コマンドについて少々触れました。  
`Kitematic`が便利だからと言っても、いざという時にコマンドが使えるようになっていると解決の幅が広がりますね。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
