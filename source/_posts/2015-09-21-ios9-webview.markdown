---
layout: post
title: "iOS9でWebViewの性能を測ろう！"
date: 2015-09-21 23:56
comments: true
categories: ios ios9 webview
---

####iOS9で使えるWebViewで各種テスト
さて、iOS8が出た時にも結果を載せましたが、iOS9でもやってみようと思います。  
iOS9では **SFSafariViewController** というものが増えましたし、UIWebViewおよびWKWebViewがSafariに対して、どの程度の性能を示すのか改めて調べておくことも重要だと思います。  

早速結果を記載していきます。  

#####HTML5の結果
[HTML5test](https://html5test.com/)の結果を記載します。  
テストは全てiOS9, iPhone6端末で実施しています。  

* UIWebView: 391
* WKWebView: 407
* SFSafariViewController: 407
* Safari: 409

上記のような結果が得られました。  
各種WebView間での差異は全て **Storage** の差です。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

#####CSS3の結果
[CSS3test](http://css3test.com/)の結果を記載します。  
テストは全てiOS9, iPhone6端末で実施しています。

* UIWebView: 55%
* WKWebView: 55%
* SFSafariViewController: 55%
* Safari: 55%

上記のようにCSS3においては差異はありませんでした。  

#####SunSpiderの結果
[SunSpider](https://www.webkit.org/perf/sunspider/sunspider.html)の結果を記載します。  
これはJavaScriptを利用したテストです。  
テストは全てiOS9, iPhone6端末で実施しています。  

* UIWebView: 1481.2 ms
* WKWebView: 351.7 ms
* SFSafariViewController: 364.5 ms
* Safari: 343.1 ms

上記のような結果が得られました。  
UIWebViewは以前もお伝えしたようにWKWebViewの約4倍程度の処理時間がかかっています。  
他は誤差の範囲で全く同じと見てよいでしょう。  

#####Octane2.0の結果
[Octane2.0](https://developers.google.com/octane/)の結果を記載します。  
これもJavaScriptを利用したテストです。  
テストは全てiOS9, iPhone6端末で実施しています。  

* UIWebView: 987
* WKWebView: 8541
* SFSafariViewController: 8112
* Safari: 8627

Octaneは点数制なので、スコアが高いほど優秀と言えます。  
UIWebViewは他と比べて、低い点数となっています。  
一方で、他の3つはほぼ誤差の範囲と見て良いでしょう。(SFSafariViewControllerが多少差がついているものの...)  

以上の結果から用途を守って正しくWebViewを使うと良いでしょう。  

* UIWebView
    * Storyboard上で配置できるので、使い方が簡単。
    * 重い処理をさせることがない(高速処理を求めない)  
* WKWebView
    * ソースコードベースで書くため、少し手間がかかる。
    * カスタマイズ性が高い＆高速処理が可能
* SFSafariViewController
    * 単なるWebサイトを表示するためだけに利用。
    * カスタマイズ性が全く不要な場合に適切
* Safari
    * アプリ外の遷移でも問題なければ、アプリからSafariを起動するのも良いでしょう。iOS9からアプリ間の遷移がスムーズになったわけですし。

と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

