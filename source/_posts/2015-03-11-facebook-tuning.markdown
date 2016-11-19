---
layout: post
title: "Facebookのモバイルチューニングの極意を読んで..."
date: 2015-03-11 15:33
comments: true
categories: facebook
---

####発展途上国への提供を見据えたモバイルチューニング技術について
さて、今回は少し趣向を変えた内容を書いてみようと思います。  
以前の記事、[iOS オールスターズ勉強会に参加しました！](http://grandbig.github.io/blog/2015/02/14/ios-allstars-study/)で通信のパフォーマンス改善の話を聞いたと書きました。  
その話の中で紹介されていた、[Facebook が語るモバイル･チューニングの極意：これで途上国のインターネットも OK！](http://urx.nu/ijSD)を読んでみて、それを自分なりにまとめてみましたので、載せたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

まず、アメリカと発展途上国を比較して、3Gの普及率とレイテンシは下図のようになっているそうです。  
![3Gの普及率とレイテンシ](/images/facebook_mobile_tuning_1.png)  

この世の中の現状に対して、Facebookアプリにダウンロードされるバイト数の大半の割合を **画像** が占めています。  
Androidアプリで言うと、実にFacebookアプリが85%, メッセンジャーが65%の割合になるようです。  

ということは、 **イメージサイズを削減すれば** 、高レイテンシなネットワークであったとしても、高品質なユーザ体験を提供することができると言えます。  

では、イメージサイズを削減するにはどんな方法があるのでしょうか？  

* クライアント側での画像のリサイズはNG(クライアント側で必要なサイズにサーバ側で事前に変換しておくこと)
* ユーザに提供する画像はサムネイルと小さいプレビュー用のもので十分(ほとんどの欲求をカバーできる)
* デバイスサイズよりも大きな画像を提供する必要はない

これは当然と言えば、当然の話ですよね。  

Facebookは特に工夫していることとして、 **webp** の活用を上げています。  
![webpのメリット](/images/facebook_mobile_tuning_3.png)  

※詳しい、画像圧縮の方式については[WebP – Webを速くするためにGoogleがやっていること ](https://html5experts.jp/jxck/2550/)を参照してください。  

また、Facebookはサーバから **レスポンス時のHTTPヘッダー内でRTT(Round-Trip Time)** を提供しています。  
その中で、Facebookは通信品質を下記の4段階に区分けしています。  

![通信品質の表](/images/facebook_mobile_tuning_4.png)  

上記を見極めた上で、

* 画像の圧縮比を増減
* 並列ネットワークリクエストの増減
* ビデオ自動再生のON / OFF
* コンテンツのプリフェッチ量の増減

といった対応を取っているとのことです。  

今回の件は、日本だけでなく、発展途上国へのサービスの提供を考えたときの参考になると感じました。  

といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
