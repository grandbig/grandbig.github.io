---
layout: post
title: "なぜか全然表示できないiOS9でのWebView(凡ミス)"
date: 2015-12-13 01:32
comments: true
categories: ios
---

####超絶凡ミスによる小１時間苦しんだiOS9でのWebView表示
今日は、久々に苦しんだ凡ミスについてメモを残しておきます。  
iOS9でハイブリットアプリを開発しようとしてハマりました。  

なぜかきちんと書いているつもりが、どうしてもWebViewが表示されない...なんてことありませんか？  
答えはiOS9だからこそハマった凡ミスでした。  

メモ程度に書いておきます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

```objective-c
import UIKit
import WebKit

class ViewController: UIViewController, WKNavigationDelegate, WKUIDelegate {
	
	var webView:WKWebView?

	override func viewDidLoad() {
		super.viewDidLoad()

		self.webView = WKWebView(frame: self.view.bounds)
		self.webview?.UIDelegate = self
		self.webview?.navigationDelegate = self

		let url = NSURL(string: "http://www.google.co.jp")
		let req = NSURLRequest(URL: url!)
		self.webView?.loadRequest(req)

		self.view.addSubview(self.webView!)
	}
}
```

さて、このままだと真っ白ページが表示されます。  
何がダメなのでしょうか？  

答えは **ATS** の設定がなかったからでした。  
URLが **http** で指定されていたため、表示がされなかったのです。  
iOS9が対象端末に入った時点で、 **SSL** は意識しておくべきですね。  

すいません。本日はホントにメモです笑  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
