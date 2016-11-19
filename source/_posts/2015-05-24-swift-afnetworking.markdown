---
layout: post
title: "Swift AFNetworkingライブラリを使ってみよう！"
date: 2015-05-24 10:53
comments: true
categories: swift ios
---

####通信系ライブラリAFNetworkingの使い方を覚えよう！
さて、本日はObjective-Cの超有名通信系ライブラリの[AFNetworking](https://github.com/AFNetworking/AFNetworking)をSwiftで使ってみようと思います。  

#####導入の仕方

まずは導入の仕方を見て行きましょう。  
CocoaPodsを使います。  
Podfileを作成し、下記の内容を記載しましょう。  

```objective-c
platform :ios, '7.0'
pod "AFNetworking", "~> 2.0"
```

これで`pod install`すればプロジェクトに **AFNetworking** が追加されるはずです。  
ここで注意したいのが、あくまでもObjective-Cで書かれたライブラリなので、Swiftで使うためにはBridgeファイルを用意する必要があります。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

1. プロジェクトにヘッダーファイルを追加します。
ProjectName-Bridging-Header.hという形式が推奨されている模様  
今回はSwiftSample-Bridging-Header.hという名前で追加しました。  
2. TARGETS > SwiftSample > Build Settings > Swift Compiler - Code Generation > Objective-C Bridging Headerにパスを指定
SwiftSample/SwiftSample-Bridging-Header.hを指定しました。
3. Bridgeファイルにライブラリを`import`する
4. ライブラリを利用するファイルにも`import`を書きましょう(Swiftの書き方で。)

下記の画像を参照ください。
![Bridgeファイルの用意](/images/swift-afnetworking.png)  

また、Bridgeファイルへの具体的な内容を書いておきます。  

```objective-c
#ifndef SwiftSample_SwiftSample_Bridging_Header_h
#define SwiftSample_SwiftSample_Bridging_Header_h

#endif

#import <AFNetworking/AFNetworking.h>
```

#####GETリクエスト
では、早速、GETリクエストを書いてみましょう。  
以前の[AFNetworking2.xでWeb-APIを叩いてみた！](http://grandbig.github.io/blog/2015/02/12/afnetworking/)でも利用した **OpenWeatherMap API** を例として使います。  

`http://api.openweathermap.org/data/2.5/weather?q=Tokyo,jp`でリクエストして、  
下記のデータを返してもらうことを想定しています。  

```objective-c

{
	base:  "stations",
	clouds: {
		all: 32
	},
	cod: 200,
	coord: {
		lat: 35.69,
		lon: 139.65
	},
	dt: 1432394992,
	id: 1850147,
	main: {
		grnd_level: 1019.97,
		humidity: 88,
		pressure: 1019.97,
		sea_level: 1024.92,
		temp: 290.99,
		temp_max: 290.99,
		temp_min: 290.99
	},
	name: Tokyo,
	sys: {
		country: "JP",
		message: 0.0386,
		sunrise: 1432323041,
		sunset: 1432374314
	},
	weather: [
		{
			description: "scattered clouds",
			icon: "03n",
			id: 802,
			main: "Clouds"
		}
	],
	wind: {
		deg: 198.001,
		speed: 5.11
	}
}

```

では、GETリクエストを投げてみましょう。  
返却されたデータの取得方法についても記載します。  

```objective-c
// GETリクエスト
let manager = AFHTTPSessionManager()
manager.GET("http://api.openweathermap.org/data/2.5/weather?q=Tokyo,jp", parameters: nil,
	success: { (operation, json) -> Void in
		// 処理が成功した場合
		let dict = json as! [String: AnyObject]
		let weatherArray = dict["weather"] as! [AnyObject]
		let weather = weatherArray[0] as! [String: AnyObject]
		let description = weather["description"] as! String
		println(description)
	}, failure: { (operation, error) -> Void in
		// エラーが発生した場合
	})
```

Objective-Cでは下記のように書いていたので、対応づけて見るとわかりやすいかもしれません。  

```objective-c
AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
[manager.GET:@"http://api.openweathermap.org/data/2.5/weather?q=Tokyo,jp", parameters: nil,
	success:^(NSURLSessionDataTask *task, id responseObject) {
		NSArray *weatherArray = [responseObject objectForKey:@"weather"];
		NSDictionary *weather = [weatherArray objectAtIndex:0];
		NSString *description = [weather objectForKey:@"description"];
		NSLog(description);
	}, failure:^(NSURLSessionDataTask *task, NSError *error) {
		// エラーが発生した場合
	}];
```

`success`と`failure`は戻り値が`void`なので、Swiftでも`-> Void`と指定しています。  
`[String: AnyObject]`は **keyがString型** で **valueがAnyObject(何でもあり)型** のDictionary型への変換で、  
`[AnyObject]`は **要素がAnyObject(何でもあり)型** のArray型への変換です。  

Alamofireとはやはり違った書き方となるので、iOSのターゲットなど要件に従って使い分けていくのが良いでしょう。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
