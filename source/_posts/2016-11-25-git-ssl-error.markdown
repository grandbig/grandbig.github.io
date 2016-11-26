---
layout: post
title: "GitのFetchでSSLエラーが出たときの対応"
date: 2016-11-25 21:46
comments: true
categories: git
---

###突如としてエラーが発生した場合の対応
先程、思いがけずエラーが出たのでメモします。  
GitでFetchしようとして下記エラーが出ました。  

```
/usr/local/Cellar/git/2.10.2/bin/git -C /Users/<username>/git/iOS/Server\ Side\ Swift/PerfectTemplate/Packages/PerfectLib.git fetch --tags origin
error: RPC failed; curl 56 SSLRead() return error -36
fatal: The remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed

error: exit(128): git -C /Users/<username>/git/iOS/Server\ Side\ Swift/PerfectTemplate/Packages/PerfectLib.git fetch --tags origin
```

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

これはどうもSSLに関するエラーが出ているようです。  
なので下記をインストールします。  

```
$ brew reinstall git --with-brewed-curl --with-brewed-openssl
```

これで再度、Gitコマンドを打ってみれば万事OKです。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>



