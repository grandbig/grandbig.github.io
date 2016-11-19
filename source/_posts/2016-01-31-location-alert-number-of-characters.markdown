---
layout: post
title: "位置情報サービスの許可アラートに表示する文字数について"
date: 2016-01-31 22:55
comments: true
categories: ios swift
---

####位置情報サービスの許可アラートに文字数制限はあるのか！？
位置情報サービスを利用するアプリの場合、位置情報サービスの利用許可を求めるアラートが表示されます。  
これはアプリで位置情報を利用するために必須の機能であり、これなくしては、`CoreLocation.framework`を通して、位置情報を利用することはできません。  

とここまではiOSエンジニアで位置情報サービスを扱ったアプリを開発した方々にとっては周知の事実かと思います。  
しかし、その許可アラートには『 **どのくらいの文字を記載することができるのか** 』まで気にする方はあまり多くないかもしれません。  

ふと気になったので、試してみました。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

Local Notification / Remote Notificationは文字数に制限があるということで以前から述べられてきました。  
例えば、下記が参考になると思います。  

* [iOSのプッシュ通知の文字数とデータサイズの制限](http://www.hangout.co.jp/blog/archives/295)
* [文字数制限を知って効果的にプッシュ通知を利用しよう！](http://iot.mb.cloud.nifty.com/iotcolumn/%E6%96%87%E5%AD%97%E6%95%B0%E5%88%B6%E9%99%90%E3%83%97%E3%83%83%E3%82%B7%E3%83%A5%E9%80%9A%E7%9F%A5)  
* [iOSのPUSH通知(APNS)の特徴・ノウハウまとめ(iOS 9まで対応)](http://qiita.com/mono0926/items/df03c61adc56934e2e7a)  

では、位置情報サービスを利用する際に表示される許可アラートに表示可能な文字数に制限はあるのでしょうか？  
iOS9ではLocal Notification / Remote Notificationの制限は4KB(4096Bytes)とのことなので、このくらいいけるのでしょうか...。  

はい。結果です。  
全角で **3938文字** まで行けました。  
これは`Info.plist`に記載できた文字数です。  
これ以上は入力しても`Info.plist`に表示されないんですね...。  
ただ、設定 > 位置情報サービス > アプリ に表示される文言はもう少し行けてるんですよ...。  

たぶんLocal Notification / Remote Notificationと同じ制限な気がしますね。  
まだまだiOS7やiOS8に対応する必要もあるでしょうし、長すぎるとそもそもユーザに対するハードルを上げることになるので全くオススメできませんよね。  

ま、文字数は気にしすぎずに、Appleが言うように極力短く的確に書くにこしたことはないと思います。  

参考までにキャプチャを貼っておきます。  

![位置情報サービス許可アラート](/images/corelocation_alert.png)  
アラート内をスクロールして全文を読むことができます。  

といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
