---
layout: post
title: "iPhoneXのステータスバーの高さ"
date: 2017-11-27 01:50
comments: true
categories: ios swift iPhoneX
---

### はじめに
今日は、今月発売されたiPhoneXについてのメモ書きです。  
iPhoneXはホームボタンがなくなり、全画面がディスプレイになっています。  
ということはステータスバーの高さも変わっているんじゃないかという話です。  

<!-- more -->

### iPhoneXのステータスバーの高さ

iPhoneXで画面キャプチャを取ってみると下図のようになります。  

![iPhoneXの画面キャプチャ](/images/iphonex_statusbar_1.png)  

これは確実にステータスバーの高さが変わっているに違いないということで調べてみました。  

```objective-c
let statusBarHeight: CGFloat = UIApplication.shared.statusBarFrame.height
print("statusBarの高さ: \(statusBarHeight)")
```

結果は以下の通りです。

iPhone8: 20.0pt  
iPhoneX: 44.0pt

ステータスバーを扱う際にはご注意を...  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
