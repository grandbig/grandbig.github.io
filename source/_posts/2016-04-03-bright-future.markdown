---
layout: post
title: "BrightFutureを使ってみよう！"
date: 2016-04-03 17:11
comments: true
categories: ios swift node
---

####SwiftでPromiseライブラリを使ってみよう
最近は更新頻度が落ちてしまっていますが、久々に更新します。  
(iPhoneSEが発表されたことですし笑)  

さて、本日はSwift版Promiseライブラリとしては割と有名な **BrightFuture** ライブラリを使ってみたいと思います。  
筆者は、新しいことに取り組む際は、JSで例を見た方がわかりやすかったりするので、Node.jsのPromiseと交えながら紹介していきたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####Promiseとは
まずは、Promiseの存在について軽く触れておきます。  
[JS Promiseの本](http://azu.github.io/promises-book/)によると、Promiseとは『非同期処理を抽象化したオブジェクトとそれを操作する仕組み』のことだそうです。  
これだけ読むと理解しがたいと思うので、筆者が感じるPromiseの良さをざっくり書くと...  

* 非同期処理を繰り返す場合でも、深いネストにならない  
* 非同期処理を繰り返す場合でも、見かけ上、直列的に処理を書ける  

ということにつきると思います。  

具体的な例については、Node.jsで説明します。  

####Node.jsでのPromiseの例
早速、見ていきます。  
Node.jsで非同期処理を定義する場合、  

```javascript
// 非同期処理を持ったメソッド
function p(msg, callback) {
	setTimeout(function() {
		callback(msg);
	}, 1000);
}
```

となります。  
このメソッドを繰り返し実行する場合、  

```javascript
p("Hello", function(msg1) {
	console.log(msg1 + ", date: " + new Date());
	p("Good Evening", function(msg2) {
		console.log(msg2 + ", date: " + new Date());
		p("Good Bye", function(msg3) {
			console.log(msg3 + ", date: " + new Date());
		});
	});
});
```

のようになります。  
ネスト深いですね...。  

これをPromiseを用いて書き直してみましょう。  
まずはメソッドの定義にPromiseを適用します。  

```javascript
function p(msg) {
	return new Promise(function(resolve) {
		setTimeout(function() {
			resolve(msg);
		}, 1000);
	});
}
```

こうすることで、呼び出し側で下記のように浅いネストで書くことができます。  

```javascript
p("Hello").then(function(msg1) {
	console.log(msg1 + ", date: " + new Date());
	return p("Good Evening");
})
.then(function(msg2) {
	console.log(msg2 + ", date: " + new Date());
	return p("Good Bye")
})
.then(function(msg3) {
	console.log(msg3 + ", date: " + new Date());
});
```

うむ。かなり浅くなりましたね。  
これをSwiftで書くとどうなるのでしょうか？  

####SwiftでのPromiseの例
Swiftでも非同期処理を定義する場合、  

```objective-c
func p(msg: String, closure: (String -> ())) {
	let queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)
	dispatch_async(queue, {
		sleep(1)
		dispatch_sync(dispatch_get_main_queue(), {
			closure(msg)
		})
	})
}
```

のように書けたりします。  
これを複数回呼び出す場合、  

```objective-c
p("Hello") { (msg1) in
	print(msg1 + ", date: \(NSDate())")
	p("Good Evening") { (msg2) in
		print(msg2 + ", date: \(NSDate())")
		p("Good Bye") { (msg3) in
			print(msg3 + ", date: \(NSDate())")
		}
	}
}
```

のようになります。  
同様にネストが深くなりますね...。  

では、Promiseでネストを改善しましょう。  
Swiftでは、OSSの[BrightFuture](https://github.com/Thomvis/BrightFutures)を利用します。  
導入はいつも通りCocoaPodsを使いました。  

1． Podfileに下記を記載  

```objective-c
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '8.0'
use_frameworks!

pod 'BrightFutures'
```

2．`pod install`を実行  

導入が完了したら、処理を書き換えていきます。  
メソッドにPromiseを適用しましょう。  

```objective-c
func p_future(msg: String) -> Future<String, NoError> {
	let promise = Promise<String, NoError>()
	let queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)
	
	dispatch_async(queue, {
		sleep(1)
		dispatch_sync(dispatch_get_main_queue(), {
			promise.success(msg)
		})
	})
	
	return promise.future
}
```

これを呼び出すときは下記のように書けます。  

```objective-c
p_future("Hello").flatMap { msg1 in
	print(msg1 + ", date: \(NSDate())")
	p_future("Good Evening")
}.flatMap { msg2 in
	print(msg2 + ", date: \(NSDate())")
	p_future("Good Bye")
}.onSuccess { (msg3) in
	print(msg3)
}
```

ネストが浅くなりましたね。  
こうして比較してみると、Node.jsとSwiftで呼び出し側の書き方はかなり似ていますね。  
Swiftだけを眺めてみてもよくわからないという方はNode.jsの方も見てみると良いと思います。  

今回はかなり基本的な例を書かせて頂きましたが、参考になれば幸いです。  
といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

