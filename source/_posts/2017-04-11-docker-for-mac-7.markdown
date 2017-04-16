---
layout: post
title: "Dockerコンテナ内のUbuntuにSwiftサーバを立てよう！"
date: 2017-04-11 19:13
comments: true
categories: docker swift
---

### はじめに
さて本日はDockerつながりで『UbuntuにSwiftサーバを立ててみたい』と思います。  
今後、ますます加熱する **Server Side Swift** を誰でも簡単に使えるように、きっとDockerを使う場面も増えてくることでしょう。  
ということで早速見ていきたいと思います。  

### Swiftのインストール方法
次の手順に従って`swift`コマンドが叩けるところまで進めてみましょう。   

１．KitematicでUbuntuを検索してDLする  

まずはDockerコンテナを作ります。

![KitematicでUbuntuをダウンロード](/images/docker_swift_1.png)  

※Ubuntuのバージョンは`Ubuntu 16.04.2 LTS`です。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

２．`Privileged mode`をONにする  

Dockerコンテナのあるある話ですが、`Privileged mode`をONにしないとコマンド利用制限かかったりするので、忘れずにしておきましょう。  

![Privileged modeをONにする](/images/docker_swift_2.png)  

３．`PortFoward`を設定する  

後々、Swiftサーバを立てた時にアクセスしたいので、Dockerにポートフォワード設定をしておきましょう。  
後ほどSwiftサーバを起動したときに`8080`にアクセスすることになるので、  

```javascript
0.0.0.0:15700->8080/tcp
```

という形でポートフォワードを設定します。  
もちろんKitematicから設定可能です。  

４．`apt-get`を最新化する  

毎度のことですが、最新化しておきます。  
まずは、Dockerコンテナにアクセスする。  

```javascript
$ docker exec -it ubuntu /bin/bash
```

```javascript
# apt-get update
# apt-get upgrade
```

５．`wget`と`vim`をインストールする  

後ほど利用するので`wget`と`vim`をインストールしておきます。  

```javascript
# apt-get install wget
# apt-get install vim
```

６．Swiftサーバの依存モジュールをインストールする  

[Swiftの公式サイト](https://swift.org/download/#using-downloads)にも書かれていますが、依存モジュールがあるのでインストールしましょう。  

```javascript
# apt-get install clang libicu-dev
```

７．Swiftパッケージをインストールする  

Swiftサーバを立てるのに必要なものは2つあります。  
それぞれ`wget`でインストールしましょう。  
今回インストールするSwiftのバージョンは **3.1** です。  

```javascript
// バイナリアーカイブの取得
# wget https://swift.org/builds/swift-3.1-release/ubuntu1604/swift-3.1-RELEASE/swift-3.1-RELEASE-ubuntu16.04.tar.gz
// 署名ファイルの取得
# wget https://swift.org/builds/swift-3.1-release/ubuntu1604/swift-3.1-RELEASE/swift-3.1-RELEASE-ubuntu16.04.tar.gz.sig
```

８．PGP鍵をインポートする  

次の手順で実施しますが、PGP(Pretty Good Privacy)の公開鍵を利用してSwiftパッケージが安全なものであると署名されたかどうかを検証します。  
その前準備としてPGP鍵をインポートします。  

```javascript
# gpg --keyserver hkp://pool.sks-keyservers.net \
      --recv-keys \
      '7463 A81A 4B2E EA1B 551F  FBCF D441 C977 412B 37AD' \
      '1BE1 E29A 084C B305 F397  D62A 9F59 7F4D 21A5 6D5F' \
      'A3BA FD35 56A5 9079 C068  94BD 63BC 1CFE 91D3 06C6'
gpg: requesting key 412B37AD from hkp server pool.sks-keyservers.net
gpg: requesting key 21A56D5F from hkp server pool.sks-keyservers.net
gpg: requesting key 91D306C6 from hkp server pool.sks-keyservers.net
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: key 412B37AD: public key "Swift Automatic Signing Key #1 <swift-infrastructure@swift.org>" imported
gpg: key 21A56D5F: public key "Swift 2.2 Release Signing Key <swift-infrastructure@swift.org>" imported
gpg: key 91D306C6: public key "Swift 3.x Release Signing Key <swift-infrastructure@swift.org>" imported
gpg: no ultimately trusted keys found
gpg: Total number processed: 3
gpg:               imported: 3  (RSA: 3)
```

９．PGP鍵で署名を検証する  

公式で説明されている通り、`key`のリフレッシュをします。  

```javascript
# gpg --keyserver hkp://pool.sks-keyservers.net --refresh-keys Swift
```

続いて、署名を検証します。  

```javascript
# gpg --verify swift-3.1-RELEASE-ubuntu16.04.tar.gz.sig
gpg: assuming signed data in `swift-3.1-RELEASE-ubuntu16.04.tar.gz'
gpg: Signature made Tue Mar 28 07:36:36 2017 UTC using RSA key ID 91D306C6
gpg: Good signature from "Swift 3.x Release Signing Key <swift-infrastructure@swift.org>"
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: A3BA FD35 56A5 9079 C068  94BD 63BC 1CFE 91D3 06C6
```

正しいSwiftパッケージであることがわかります。  

１０．パッケージを展開する  

パッケージは圧縮されているので展開しましょう。  

```javascript
# tar xzf swift-3.1-RELEASE-ubuntu16.04.tar.gz
```

１１．パスを通す  

さて、DLしたパッケージに対して検索パスを設定する必要があります。  
Dockerコンテナには環境設定ファイルとして`.dockerenv`があるのでこちらにパスを記載します。  

```javascript
// 解凍フォルダの名前が長いので名称を変更する
# mv swift-3.1-RELEASE-ubuntu16.04 swift

// ファイルを開く
# vi .dockerenv

// 下記のようにパスを書く
export PATH=/swift/usr/bin:"${PATH}"

// 環境設定を更新する
# source .dockerenv

// 検索パスが通ったことを確認する
# which swift
/swift/usr/bin/swift
# swift --version
Swift version 3.1 (swift-3.1-RELEASE)
Target: x86_64-unknown-linux-gnu
```

１２．必要なモジュールをインストールする  

このままで`swift`コマンドを叩いても下記のようにエラーが表示されます。  

```javascript
/swift/usr/bin/lldb: error while loading shared libraries: libedit.so.2: cannot open shared object file: No such file or directory
```

どうやら必要なモジュールが足りないようですね...  
ということで下記をインストールしましょう。  

```javascript
# apt-get install libedit-dev
# apt-get install libpython-dev
# apt-get install libxml2-dev
```

これで`swift`コマンドを実行できるようになります。  
ついでに`Hello World`まで出力してみます。  

```javascript
# swift
Welcome to Swift version 3.1 (swift-3.1-RELEASE). Type :help for assistance.
  1> print("Hello, world!")
Hello, world!
```

### PerfectでSwiftサーバを立てよう！
今回もSwiftサーバを立てるために[Perfect](https://github.com/PerfectlySoft/Perfect)を利用します。  
下記手順に従って進めましょう。  

１．`git`をインストールする  
まずは`Perfect`を`git`経由でダウンロードする必要があるので`git`をインストールしましょう。  

```javascript
# apt-get install git
```

２．`PerfectTemplate`をダウンロードする  
簡潔にSwiftサーバを立てたいので`PerfectTemplate`をダウンロードします。  

```javascript
# git clone https://github.com/PerfectlySoft/PerfectTemplate.git
```

３．必要モジュールをインストールする  
PerfectのGet Startedに書かれている通り、次の必要なモジュールをインストールします。  

```javascript
# apt-get install openssl libssl-dev uuid-dev
```

４．不足モジュールをインストールする  
このままだと`swift build`実行しても下記エラーが出てしまいました。  

```javascript
error while loading shared libraries: libcurl.so.4: cannot open shared object file: No such file or directory
```

上記解決のためにモジュールをインストールしましょう。  

```javascript
# apt-get install libcurl4-openssl-dev
```

５．`swift build`を実行する  
さて、ビルドしましょう。  

```javascript
# cd PerfectTemplate
# swift build
```

６．Swiftサーバを起動する  
後は下記コマンドでSwiftサーバを起動しましょう。  

```javascript
# .build/debug/PerfectTemplate
[INFO] Starting HTTP server localhost on 0.0.0.0:8080
[INFO] Starting HTTP server localhost on 0.0.0.0:8181
```

![Swiftサーバにアクセス](/images/docker_swift_3.png)  

### まとめ
さていかがでしたでしょうか？  
筆者個人としては、途中で何回かハマったものの、DockerコンテナのUbuntuにSwiftサーバを立てられたことに満足できました。  
今回はサーバを立てるという基本的なものでしたが、今度はWeb APIを作成して、iOSアプリからアクセスしてみたりなど遊んでみたいと思います。  

と言ったところで本日はここまで。  

参考:  

* [Getting Started with Swift on Linux](https://www.twilio.com/blog/2015/12/getting-started-with-swift-on-linux.html)  
* [Ubuntu上でSwiftサーバーサイドフレームワークのPerfectを動かしてみた](http://qiita.com/yonell/items/79bf4ee3cd65f69903ec)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
