---
layout: post
title: "今更だけど使ってみようSwiftyJson"
date: 2016-03-06 22:34
comments: true
categories: ios swift
---

####SwiftyJsonの便利さを改めて確かめてみる
さて、筆者の個人的な諸事情により、なかなか時間が取れないため、  
完全に新しいことを書けないでいる今日このごろです。  
今更ながら書くか否か迷ったのですが、  
折角、習慣化しているブログ投稿をSTOPさせたくもないので、  
Swift屋にとっては当たり前となっているであろう[SwiftyJson](https://github.com/SwiftyJSON/SwiftyJSON)について書こうと思います。  
何を隠そう、筆者は今までSwiftyJsonの存在は把握していたものの、  
自身で変換して書くことに拘っていたので触っていませんでした。  
がしかし、食わず嫌いは如何なものかということで触ってみることにしました。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####CocoaPodsでインストール
まずは、インストールです。  
`CocoaPods`, `Carthage`, `Swift Package Manager`に対応した良心的なライブラリです。  
筆者はいつも通り`CocoaPods`を使います。  
今回も`Open Weather Map API`をサンプルに利用したいので[Alamofire](https://github.com/Alamofire/Alamofire)も合わせてインストールします。  

```objective-c
platform :ios, '8.0'
use_frameworks!

target 'SwiftyJsonSample' do
	pod 'Alamofire', '~> 3.0'
	pod 'SwiftyJSON', :git => 'https://github.com/SwiftyJSON/SwiftyJSON.git'
end
```

####SwiftyJsonで何が便利になったのか
さて、早速、`SwiftyJson`を使うことで何が便利になるのか見てみたいと思います。  
まずは、便利にする前のベースのコードをご覧下さい。  

```objective-c
Alamofire.request(.GET, "http://api.openweathermap.org/data/2.5/weather?APPID=<あなた自身のAPPID>",
	parameters:["q":location]).responseJSON { (response) -> Void in
		if response.result.isSuccess {
			guard let value = response.result.value else {
				// 値の取得に失敗した場合
				return
			}
			guard let weatherArray:AnyObject? = value["weather"] else {
				// 値の取得に失敗した場合
				return
			}
			guard let weather = weatherArray?[0] else {
				// 値の取得に失敗した場合
				return
			}
			guard let description = weather["description"] as? String else {
				// 値の取得に失敗した場合
				return
			}

			print(description)
		}
}
```

前回の記事で紹介した`guard statement`を使うことでネストが浅くなっていますね。  
これはわかりやすさのために、バラけて書いたのですが、  
元々 **description** の値が欲しいのであれば、  

```objective-c
Alamofire.request(.GET, "http://api.openweathermap.org/data/2.5/weather?APPID=<あなた自身のAPPID>",
	parameters:["q":location]).responseJSON { (response) -> Void in
		if response.result.isSuccess {
			guard let description = response.result.value?["weather"]??[0]["description"] else {
				// 値の取得に失敗した場合
				return
			}
			print(description!)
		}
}
```

とすればかなり短く書けます。  
しかしながら、素人にとって **オプショナル型** や **アンラップ** は相当わかりにくいと言えます。  
これが`SwiftyJson`を利用することで細かい上記のことを気にすることなく、  
WEBで扱うようにJSONを扱えます。  
もちろんSwiftでは重要なことなので、  
[どこよりも分かりやすいSwiftの"?"と"!"](http://qiita.com/maiki055/items/b24378a3707bd35a31a8)を参考に正しく理解しましょう。  

さて、`SwiftyJson`を使うと次のように書けます。  

```objective-c
Alamofire.request(.GET, "http://api.openweathermap.org/data/2.5/weather?APPID=<あなた自身のAPPID>",
	parameters:["q":location]).responseJSON { (response) -> Void in
		if response.result.isSuccess {
			guard let data = response.result.value else {
				return
			}
			let json = JSON(data)
			guard let description = json["weather"][0]["description"] else {
				return
			}
			print(description)
		}
}
```

多少行数は増えているものの、  
**オプショナル型** の **アンラップ化** を気にせず書けていることがわかると思います。  
そう、まるでWEBでJSONを扱うかのように...。  

オプショナル型の値が内部にオプショナル型の値を持っているなど、  
`??`と書く状況を作りかねないことがあっても`SwiftyJson`では考えなくて良いんですね。  
う〜ん。便利。  

他にもたくさんの使い道があるため機会があれば積極的に利活用していこうと思います。  
また、これまでスルーしてきた他の便利ライブラリについてもちょくちょくその便利さを追求していこうと思います。  

と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

