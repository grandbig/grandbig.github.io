---
layout: post
title: "SwiftでNSDateFormatterを使ってNSDateからStringに変換してみる！"
date: 2016-02-19 23:17
comments: true
categories: ios swift
---

####NSDateFormatterとは
さて、本日は`NSDateFormatter`について見ていきます。  
`NSDateFormatter`は

* `NSDate`から`String`へ変換する際に指定するフォーマットオブジェクト  
* `String`から`NSDate`へ変換する際に指定するフォーマットオブジェクト

として使ったりします。  
(他にも`NSCalendar`等々で利用します。)  
基本的な変換は下記のようにできます。  

```objective-c
func convertStringFromDate(date: NSDate) -> String {
	let dateFormatter = NSDateFormatter()
	dateFormatter.dateFormat = "yyyy-MM-dd HH:mm:ss.SSSSSS"

	return dateFormatter.stringFromDate(date)
}
```

続いて、よく利用する各種設定について見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####NSTimeZoneについて
`NSTimeZone`はグリニッジ標準時と各地域の標準時の時差情報を持ったクラスです。  
このクラスを利用することで、欲しい時差情報を含んだ日付を取得できます。  

例えば下記のような形です。  

```objective-c
func convertStringFromDate(date: NSDate) -> String {
	let dateFormatter = NSDateFormatter()
	dateFormatter.timeZone = NSTimeZone.defaultTimeZone()
	dateFormatter.dateFormat = "yyyy-MM-dd HH:mm:ss.SSSSSS"

	return dateFormatter.stringFromDate(date)
}
```

`NSTimeZone`には`systemTimeZone`, `defaultTimeZone`, `localTimeZone`, 直接地域を指定する方法があります。  

* `systemTimeZone`:  
設定 > 一般 > 日付と時刻 で設定されている地域のタイムゾーンを返却する  
* `defaultTimeZone`:  
アプリで設定された地域のタイムゾーン  
アプリで設定されていない場合は`systemTimeZone`を返却する  
* `localTimeZone`:  
`defaultTimeZone`への参照値を返却する  
* 直接タイムゾーンを指定:  
`GMT`, `UTC`, `ja_JP`, `en_US`などの地域を指定することで、指定地域のタイムゾーンを返却する  

仮に、それぞれのタイムゾーンの設定をして文字列を出力してみると、  

```objective-c
systemTimeZone: 2016-02-19 23:27:37.550000
defaultTimeZone: 2016-02-19 23:27:37.550000
localTimeZone: 2016-02-19 23:27:37.550000
GMT: 2016-02-19 14:27:37.551000
UTC: 2016-02-19 14:27:37.551000
ja_JP: 2016-02-19 23:27:37.551000
```

といった感じです。  

####NSLocaleについて
`NSLocale`は言語や分化などの情報を持ったクラスです。  
このクラスを利用することで、欲しい言語や分化の地域情報を含んだ日付を取得できます。  

利用方法は下記のようになります。  

```objective-c
func convertStringFromDate(date: NSDate) -> String {
	let dateFormatter = NSDateFormatter()
	dateFormatter.locale = NSLocale.currentLocale()
	dateFormatter.dateFormat = "yyyy-MM-dd HH:mm:ss.SSSSSS"

	return dateFormatter.stringFromDate(date)
}
```

* `systemLocale`:  
設定 > 一般 > 日付と時刻 で設定されている地域の情報を返却する  
* `currentLocale`:  
現在のユーザで設定されている地域の情報を返却する  
マルチユーザ環境下であれば、違いが出ると思います。  

地味に文字列変換って面倒だったりするんですよね...。  
とは言え、アプリを世界対応させる場合は、  
ユーザの設定している **日付と時刻** に注意を配ることが大切です。  

と言ったところで本日はここまで。  

参考:  

* [iOSの日付処理まとめ](http://aqubiblog.blogspot.jp/2012/11/ios.html)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
