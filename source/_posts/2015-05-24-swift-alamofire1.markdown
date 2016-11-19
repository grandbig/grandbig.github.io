---
layout: post
title: "Swift Alamofireライブラリを使ってみよう！"
date: 2015-05-24 00:40
comments: true
categories: swift ios
---

####通信系ライブラリAlamofireの使い方を覚えよう！
さて、本日は **Alamofire** ライブラリの使い方を勉強します。  
[Alamofire](https://github.com/Alamofire/Alamofire)はObjective-Cの超有名通信系ライブラリである[AFNetworking](https://github.com/AFNetworking/AFNetworking)のSwift版と思えばよいでしょう。  

#####導入の仕方

まずは導入の仕方から見て行きましょう。  
CocoaPodsを使います。  
Podfileを作成し、下記の内容を記載しましょう。  

```objective-c
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '8.0'
use_frameworks!

pod 'Alamofire', '~> 1.2'
```

これで`pod install`すればプロジェクトにAlamofireが追加されるはずです！  
後は利用したいswiftファイルで`import Alamofire`しましょう。  
SwiftライブラリをCocoaPodsでインストールすると対象iOSが8.0以上である必要が出てくるので、そこだけ注意しましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

#####GETリクエスト(パターン１)

続いて、GETリクエストのメソッドを書いてみます。  
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
幾つか書き方があるのですが、まずは１つ目の書き方です。  

```objective-c
// GETリクエスト
Alamofire.request(.GET, "http://api.openweathermap.org/data/2.5/weather", parameters:["q":"Tokyo,jp"]).response { (request, response, data, error) in
	
	if let data = data as? NSData {
		// AnyObject型をNSData型に変換できた場合に通過
		// NSData型をJSONの形に変換して、AnyObject?型の変数に格納
		var obj: AnyObject? = NSJSONSerialization.JSONObjectWithData(data, options: NSJSONReadingOptions.AllowFragments, error: nil)

		if let weatherArray = obj["weather"] as? [AnyObject] {
			// objからweatherフィールドを取得。その際、Array型に変換できた場合に通過
			if let weather = weatherArray[0] as? [String: AnyObject] {
				// NSArray型のweatherArrayから要素0番目を取得。その際、Dictionary型に変換できた場合に通過
				if let description = weather["description"] as? String {
					// NSDictionary型のweatherからdescriptionフィールドを取得。その際、String型に変換できた場合に通過
					println(description)
				}

				// NSDictionary型のweatherからdescriptionを指定して取得
				var description2:String? = weather["description"] as? String
				println(description2)
			}
		}
	}
}
```

注意したいのは、responseで返却される`data`は **NSData型** ではなく、 **AnyObject?型** だということです。  
そのため、直接`NSJSONSerialization.JSONObjectWithData`の１つ目の引数に`data`を指定するとNSData型ではないことを示唆するエラーが出ます。  
なので、 **NSData型** に変換する必要があります。  

そして、NSData型のままでは各要素を取り出せないので、 **(JSONの形をした)AnyObject?型** に変換させましょう。  
(Dictionary型に変換しても良いのですが、既にJSONの形に変換されているので十分でしょう。)  
後は各フィールドの型に従って変換して取り出していくイメージで書きましょう。  

覚えておきたいのは、  

```objective-c
if let 定数名 = 要素 as? 定数の型 {
	// 指定した定数の型に変換できた場合に、この中を通過する(変換できなかった場合はnilを入れようとするので、この中を通過しない)
}
```

という変換の書き方です。  
Swiftの場合、JSONの取り扱いで思った以上にコードが長くなり、ネスト化してしまいます。  
それを防ぐため&JavaScriptのような書き方を意識したライブラリがたくさん出ています。  
近いうち使ってみようと思います。  

また、変換時に`[AnyObject]`や`[String: AnyObject]`と書いているのは、  
前者が`Array型`に変換していて、後者は`Dictionary型`に変換しています。  
Swiftではどんな要素を含むArray型なのかを示し、どんなkeyでどんなvalueなのかをDictionary型として明示的に示す必要があります。  

もし、Objective-Cの`NSArray型`や`NSDictionary型`に変換する場合は中身の形式を考えずに`as`の後ろに続けて書けばOKです。  

#####GETリクエスト(パターン２)
とは言え、少しネストが深いので、別のresoponse形式のパターンで試してみましょう。  

```objective-c
// GETリクエスト
Alamofire.request(.GET, "http://api.openweathermap.org/data/2.5/weather?q=Tokyo,jp").responseJSON { (_, _, JSON, _) in
	if let weatherArray = JSON!["weather"] as? [AnyObject] {
		// JSONからweatherフィールドを取得。その際、Array型に変換できた場合に通過
		if let weather = weatherArray[0] as? [String: AnyObject] {
			// NSArray型のweatherArrayから要素0番目を取得。その際、Dictionary型に変換できた場合に通過
			if let description = weather["description"] as? String {
				// NSDictionary型のweatherからdescriptionフィールドを取得。その際、String型に変換できた場合に通過
				println(description)
			}

			// NSDictionary型のweatherからdescriptionを指定して取得
			var description2:String? = weather["description"] as? String
			println(description2)
		}
	}
}
```

これは、レスポンスメソッドに`responseJSON`を指定することで、  
1つ目に上げた`AnyObject?型 ⇒ NSData型`への変換と`NSData型 ⇒ JSONの形をしたAnyObject?型`への変換は不要となります。  
既に`responseJSON`の中でやってくれています。  
少しネストが浅くなりましたね！  

#####GETリクエスト(パターン３)
まだ、ネストが深いって？  
よし！思い切ってネストをなくしましょう！

```objective-c
// GETリクエスト
Alamofire.request(.GET, "http://api.openweathermap.org/data/2.5/weather?q=Tokyo,jp").response { (request, response, data, error) in
	// AnyObject?型をNSData型に変換。さらにJSONの形をしたNSDictionary型に変換。
	let json = NSJSONSerialization.JSONObjectWithData(data as! NSData, options: NSJSONReadingOptions.MutableContainers, error: nil) as! [String: AnyObject]
	let weatherArray = json["weather"] as! [AnyObject]
	let weather = weatherArray[0] as! [String: AnyObject]
	let description = weather["description"] as! String
	println(description)
}
```

う〜ん。ネストは確かになくなったのですが、Unwrapの手法を **Implicitly Unwrapped Optional** にしたため、nilが発生した段階でエラーが発生して落ちます...。  
値がない場合には空オブジェクトを返すAPIである場合には問題ないと思います。  

だんだんとSwiftの基礎含めて理解が進んできた気がします。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
