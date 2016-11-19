---
layout: post
title: "Bolts-Swiftを使ってみよう！"
date: 2016-05-15 22:34
comments: true
categories: ios swift
---

####SwiftでBolts-Swiftライブラリを使ってみよう
これまで、`BrightFuture`, `PromiseKit`とPromiseライブラリについて見てきましたが、本日はFacebook製のPromiseライブラリである[Bolts-Swift](https://github.com/BoltsFramework/Bolts-Swift)を試してみたいと思います。  

`Bolts`はこれまでもObjective-Cコードで利用されてきた有名ライブラリですが、  
2016-03-18にSwift版 v1.0.0(`Bolts-Swift`) が公開されました。  
※ Objective-C版は[Bolts-ObjC](https://github.com/BoltsFramework/Bolts-ObjC)です。  
※ 本記事執筆時点ではv1.1.0が最新となっています。  

Facebookのエンジニアが開発しているため、Facebook SDKを利用する場合でも内部で`Bolts-ObjC`が利用されています。  
現在、Facebook SDKがObjective-Cで書かれているため、今後Swift化される際に、`Bolts-Swift`が使われることになる可能性が高いと思います。  
知っておいて損はないでしょう。  

ということで早速、使い方を見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####Bolts-Swiftのインストール
CocoaPodsでインストールします。  

**1．Podfileに下記を記載**  

```objective-c
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '8.0'
use_frameworks!		// (1)

target "BoltsSwiftSample" do	// (2)
	pod 'Bolts-Swift'
end
```

`Bolts-Swift`の場合は(1)と(2)が必須書式となります。  
(書かない場合、インストール時にErrorが発生します。)  

**2．`pod install`を実行**  

**3．処理の実装**  

ライブラリのインストールが完了したら、処理を書いていきましょう。  

`Bolts-Swift`を適用したメソッドを用意します。  
※ 非同期処理を含むサンプルメソッドです。  

```objective-c
func p_task(msg: String) -> Task<String> {
	let taskCompletionSource = TaskCompletionSource<String>()
	let queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)
	dispatch_async(queue, {
		sleep(1)
		dispatch_sync(dispatch_get_main_queue(), {
			print(msg)
			taskCompletionSource.setResult(msg)
		})
	})
							        
	return taskCompletionSource.task
}
```
このように定義することで、下記のように呼び出すことが可能となります。  

```objective-c
let bt = BoltsTask()
bt.p_task("Hello").continueOnSuccessWith { (msg1) -> Task<String> in
	return bt.p_task("Good Evening")
}.continueOnSuccessWith { (msg2) -> Task<String> in
	return bt.p_task("Good Bye")
}
```

これを実行すれば、  

```objective-c
Hello
Good Evening
Good Bye
```

と結果が得られます。  
以上のように『ネストを浅く』、『非同期処理を直列的に書く』ことができます。  

`BrightFuture`や`PromiseKit`を試しに使ってきたことで思ったのが、同様の書式で書けることが多いということです。  
一見、GitHub上のReadmeを読んでもわからないと思うライブラリがあった際には、用途が似ているライブラリのReadmeを見てみるのも1つの手段かもしれません。  

といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
