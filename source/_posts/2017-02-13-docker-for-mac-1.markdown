---
layout: post
title: "Docker for Macをインストールしよう！"
date: 2017-02-13 00:06
comments: true
categories: docker
---

###はじめに
本日は **Docker for Mac** についてメモ程度に学んだことを書き残しておきたいと思います。  
`Docker for Mac`とはMacアプリとしてインストールできるDockerです。  
今までは、MacでDockerを利用するために`Docker Toolbox`と`Virtual Box`をインストールする必要がありましたが、`Docker for Mac`の登場により、いろいろとやってくれそうな感があります。  
(筆者もDocker自体にそんなに詳しいわけではないので、1つずつ見ていきたいと思います。)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

###Docker for Macのインストール
簡単かと思いきや、なぜか詰まることもあるので書きます。  
インストールは[Docker公式ページ](https://docs.docker.com/docker-for-mac/)から可能です。  
公式ページを見ると、 **Download Docker for Mac** という文字が見つかります。  
`Stable channel`と`Beta channel`の2つがあるのですが、筆者は最終的に`Beta channel`をインストールしました。  
なぜなら、`Stable channel`では`docker run hello-world`が実行エラーになったためです...  

![Docker for Mac Betaのダウンロード](/images/docker_1.png)  

インストールが完了すると、macのツールバーにDockerのマークが出現します。  

![ツールバーにDockerマーク](/images/docker_2.png)  

Dockerが正しくインストールされたので下記コマンドでバージョンを確認してみましょう。  

```javascript
$ docker --version
Docker version 1.13.1, build 092cba3

$ docker-compose --version
docker-compose version 1.11.1, build 7c5d5e4

$ docker-machine --version
docker-machine version 0.9.0, build 15fd4c7
```

続いて、Dockerが期待通りに動くことを確認するために下記コマンドを叩いてみましょう。  
まずはDockerのバージョン確認です。  

```javascript
$ docker version
Client:
 Version:      1.13.1
 API version:  1.26
 Go version:   go1.7.5
 Git commit:   092cba3
 Built:        Wed Feb  8 08:47:51 2017
 OS/Arch:      darwin/amd64

Server:
 Version:      1.13.1
 API version:  1.26 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   092cba3
 Built:        Wed Feb  8 08:47:51 2017
 OS/Arch:      linux/amd64
 Experimental: true
```

`Client`と`Server`の両方のバージョンが確認できるようです。  
(中身があまり変わらないように見えるけど...)  

続いて、ローカルに存在するDockerコンテナの確認です。  

```javascript
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

何もないと当然ですが、上記のように何も表示されません。  
最も初歩であろう`hello-world`コマンドです。  

```javascript
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
78445dd45222: Pull complete
Digest: sha256:c5515758d4c5e1e838e9cd307f6c6a0d620b5e07e6f927b07d05f6d12a1ac8d7
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://cloud.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

ローカルに存在しないとダウンロードを始めます。  
冒頭に述べた`Stable channel`では、ここで全くダウンロードができませんでした。  
何はともあれ、これで **Docker for Mac** のインストール完了です。  

###Docker for Macを使ってUbuntuをインストールしてみよう
先程インストールした`hello-world`の結果の中に`To try something more ambitious, you can run an Ubuntu container with:`とありました。  
興味本位ですが、コマンド叩いてみましょう。  

```javascript
$ docker run -it ubuntu bash
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
8aec416115fd: Pull complete
695f074e24e3: Pull complete
946d6c48c2a7: Pull complete
bc7277e579f0: Pull complete
2508cbcde94b: Pull complete
Digest: sha256:71cd81252a3563a03ad8daee81047b62ab5d892ebbfbf71cf53415f29c130950
Status: Downloaded newer image for ubuntu:latest
root@3e95790a60cb:/#
```

これで`Ubuntu`がインストールできたようです。  
しかも`Ubuntu`の中に入れてますね！  

本当に`Ubuntu`インストールできたか確かめるためにもコマンドを叩いてみましょう。  

```javascript
root@3e95790a60cb:/# cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04.1 LTS"

root@3e95790a60cb:/# arch
x86_64

root@3e95790a60cb:/# uname -a
Linux 3e95790a60cb 4.9.8-moby #1 SMP Wed Feb 8 09:59:13 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
```

バッチリですね！！  
(正直な話、`uname -a`とか数年ぶりに叩いた気がして懐かしい...笑)  

これで後はもう様々な便利なものをインストールして楽しめそうですね。  

###Kitematicの紹介
メニューバーのDockerマークをクリックすると、Docker for Macのメニュー項目が表示されます。  
その中に`Kitematic`という聞きなれないものがあると思います。  
こちらをインストールするとターミナルからコマンドを叩かずとも、GUIからクリック操作で様々なことが可能になります。  

![Dockerのメニュー](/images/docker_3.png)  

起動してログインすると(初めはアカウントがないと思うので作成しましょう。)、下記のような画面が表示されます。  

![Kitematic](/images/docker_4.png)  

最早、見るだけで心躍りますよね！？  
様々なものをインストールして楽しめそうな気しかしません！  

因みに、下記のようにコンテナの起動などが可能です。  

![KitematicでDockerコンテナ起動](/images/docker_5.png)  

###まとめ
さて、今回は珍しく基盤よりなDockerで遊んでみました。  
この付近の知識はあって困ることはないですし、Dockerかなり一般的になってきているので引き続きブログの題材として取り上げていきたいと思います。  
と言ったところで本日はここまで。  


<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
