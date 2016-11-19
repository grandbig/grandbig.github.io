---
layout: post
title: "CROSSWALKの性能を確かめよう！"
date: 2015-04-12 16:50
comments: true
categories: android crosswalk webview
---

####CROSSWALKは果たして優秀なのか！？
以前、CROSSWALKについて紹介させて頂きました。  
メリット/デメリットはいろいろなサイトで紹介されていますが、実際に性能はどうなのか気になりますよね？  
そこで今回は幾つかの端末を用いて試験をしてみました。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

#####Sunspiderでベンチマーク
試験に利用した端末は下記3つです。  

* Android4.4.2 / AQOUS 305SH
* Android4.2.2 / AQOUS 303SH

これら端末で下記4つのWebの性能を確かめました。  

* CROSSWALK
* 標準のWebView
* 標準のブラウザ
* Chrome

早速ですが、結果を載せます。  

![Android4.4.2, AQOUS 305SH, Sunspider結果](/images/crosswalk_sunspider_305sh.png)  
![Android4.2.2, AQOUS 303SH, Sunspider結果](/images/crosswalk_sunspider_303sh.png)  

結果を見ると、処理速度だけで言えば、必ずしもCROSSWALKが優秀とは言えないことがわかります。  
しかし、CROSSWALKの良さは処理速度に限ったことではありません。  
何と言っても最新のChromiumと同じHTML, CSSの対応度合いが肝です。  
では続けて、HTMLとCSSに関して試験してみましょう。  

#####HTML5のテスト
まずは、HTML5の対応度合いを見て行きましょう。  
計測に使うのはもちろん[HTML5test](https://html5test.com/)です。  
HTML5testは550点満点なので、数値が550に近いほど優秀と言えます。  

![Android4.4.2, AQOUS 305SH, HTML5Test結果](/images/crosswalk_html5test_305sh.png)  
![Android4.2.2, AQOUS 303SH, HTML5Test結果](/images/crosswalk_html5test_303sh.png)  

これはさすがの結果と言えるでしょう。  
Chromeの方が高かったりしますが、標準のWebViewや標準のブラウザに比べれば遥かに優秀であると言えます。  

#####CSS3のテスト
次に、CSS3の対応度合いを見て行きましょう。  
計測に使うのはもちろん[The CSS3 Test](http://css3test.com/)です。  
The CSS3 Testは100%満点の%で表現されます。  

![Android4.4.2 AQOUS 305SH, CSS3Test結果](/images/crosswalk_css3test_305sh.png)  
![Android4.2.2 AQOUS 303SH, CSS3Test結果](/images/crosswalk_css3test_303sh.png)  

こちらもCROSSWALKがメリットとなる結果と言えるでしょう。  
Chromeと同じ対応度合いであったので、全く問題がないと言えると思います。  

#####まとめ
以上の結果を見ると、処理速度が気になるものの、WebSocketやWebGL, WebRTCなどに対応した上で、ターゲットにAndroid4.0〜4.3を含めたいのであれば、CROSSWALKは入れるべきと言えるでしょう。  
Android5.0以上がほとんどを占める世の中になったとしたら、何も考えずに標準のWebViewを使えば良いのかもしれないですが、まだまだ先の話でしょう。  
と言ったところで、本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
