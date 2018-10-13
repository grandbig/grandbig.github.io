---
layout: post
title: "PromiseKitを使ってみよう！"
date: 2016-04-09 23:07
comments: true
categories: ios swift promise
---

####SwiftでPromiseライブラリを使ってみよう
さて、前回から引き続きSwiftでのPromiseライブラリについて見ていきたいと思います。  
今回はPromiseライブラリのSwift界隈で有名な **PromiseKit** について見ていきます。  

####PromiseKitのインストール
早速、 **PromiseKit** の導入について紹介します。  
筆者の場合、Xcode7.3を利用しているため、Swift2.2を扱うことになります。  
[GitHub](https://github.com/mxcl/PromiseKit)で紹介されている下記の導入方法では **2.2.1** がインストールされます。  
しかし、このバージョンはSwift2.2に対応していないため、エラーが発生します。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

```objective-c
use_frameworks!

pod "PromiseKit", "~>2.0"
```

[PromiseKitの公式ドキュメントページ](http://promisekit.org/)を見ると、Newsの項目でバージョン **3.1.0** でSwift2.2に対応したと書かれています。  
そこで、今回は、  

```objective-c
use_frameworks!

pod "PromiseKit", "3.1.0"
```

とすることで、Xcode7.3でも問題なく、PromiseKitを扱えるようにしました。  

####PromiseKitで非同期処理を直列に書いてみよう
前回のBrightFutureで紹介した処理をPromiseKitで書いてみようと思います。  
まずは、非同期処理をPromiseKitを使って定義します。  

```objective-c
func p(msg: String) -> Promise<String>{

	let promise = Promise<String> { (fulfill, reject) in
		let queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)
		dispatch_async(queue, {
			sleep(1)
			dispatch_sync(dispatch_get_main_queue(), {
				print(msg)
				fulfill(msg)
			})
		})
	}

	return promise
}
```

では、これを呼び出してみましょう。  

```objective-c
p("Hello").then { (msg1) -> Promise<String> in
	return p("Good Evening")
}.then { (msg2) -> Promise<String> in
	return p("Good Bye")
}
```

ネストが浅い状態で複数の非同期処理を書くことができました。  
また、BrightFutureと大きな違いはなく書くことできました。  

####PromiseKitで非同期処理を並列に書いてみよう
続いて、複数の非同期処理を並列に実行する場合を考えてみます。  
順番に処理をする必要がなく、処理が完了したものから反映していきたい場合などに使いますね。  

非同期処理自体は先程と同じものを使います。  
では、並列に処理を呼び出してみましょう。  

```objective-c
// 実行したい非同期処理を定義
let p1 = ps.p_promiseKit("Method1")
let p2 = ps.p_promiseKit("Method2")
let p3 = ps.p_promiseKit("Method3")

// まとめて非同期処理を実行
when([p1, p2, p3]).then { (result: [String]) -> AnyPromise in
	return AnyPromise(bound: Promise())
}
```

先に書いた直列処理では、1秒間隔でログが出力されたかと思います。  
ここでは同時に実行したため、ほぼ同時に3つのログが出力されたはずです。  

さて如何だったでしょうか？  
SwiftでもPromiseライブラリはたくさん出てきました。  
[BrightFuture](https://github.com/Thomvis/BrightFutures), [PromiseKit](https://github.com/mxcl/PromiseKit), [SwiftTask](https://github.com/ReactKit/SwiftTask)のいずれを利用するかは開発者の判断になるかと思います。  
個人的にはPromiseKitはGitHub上に似た名称のブランチがたくさんあるのをどうにかして欲しいなと思ってたりします笑  

また、時間があるときにPromiseを使って何かやりたいと思います。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
