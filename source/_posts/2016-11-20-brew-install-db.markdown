---
layout: post
title: "HomebrewでMongoDBをインストール"
date: 2016-11-20 00:10
comments: true
categories: mongodb
---

###PC移行したことで発生したHomebrew対応し直し
さて、本日はMacbook Proを購入したことでブログ作成環境を移行する必要があったので、その際に気づいたことをメモしておきます。

これまではMongoDBやMySQLなどのDB系はインストールした後に、次回、PCログイン時に自動起動させるために手動でデーモン起動設定をする必要がありました。  
しかし、今回、筆者が **Homebrew** で上記をインストールしてみたところ、新たな記述が出力されるようになっていました。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!--more-->

```
// MongoDBの例
$ brew install mongodb
==> Downloading https://homebrew.bintray.com/bottles/mongodb-3.2.11.sierra.bottle.tar.gz
######################################################################## 100.0%
==> Pouring mongodb-3.2.11.sierra.bottle.tar.gz
==> Caveats
To have launchd start mongodb now and restart at login:
  brew services start mongodb
Or, if you don't want/need a background service you can just run:
  mongod --config /usr/local/etc/mongod.conf
==> Summary
🍺  /usr/local/Cellar/mongodb/3.2.11: 18 files, 245.4M
```

上記が実際に出力された記述になります。  
筆者が注目したのは、 **brew services start mongodb** の部分です。  
これを叩けば、デーモン起動設定がされるのでは！？と思った次第です。  

では実際にコマンドを叩いてみましょう。  

```
$ brew services start mongodb
==> Tapping homebrew/services
Cloning into '/usr/local/Homebrew/Library/Taps/homebrew/homebrew-services'...
remote: Counting objects: 10, done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 10 (delta 0), reused 6 (delta 0), pack-reused 0
Unpacking objects: 100% (10/10), done.
Tapped 0 formulae (37 files, 46.8K)
==> Successfully started `mongodb` (label: homebrew.mxcl.mongodb)
```

起動しているか下記コマンドで確認してみます。  

```
$ ps aux | grep mongo
takahiro      5468   0.2  0.2  2584528  25672   ??  S    11:24PM   0:00.22 /usr/local/opt/mongodb/bin/mongod --config /usr/local/etc/mongod.conf
takahiro      5472   0.0  0.0  2432804   1924 s000  S+   11:24PM   0:00.00 grep mongo
```

これで既に起動していることを確認できました。  
すごく便利になりましたね！

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
