---
layout: post
title: "Swift2.xでのシングルトンパターンとクラスメソッド / インスタンスメソッド"
date: 2016-02-06 11:41
comments: true
categories: ios swift
---

####シングルトンを使用したクラスでクラスメソッドとインスタンスメソッドを定義する
本日は、  

- Swift2.xでのシングルトンパターンのクラスの定義の仕方  
- そのクラスにおけるクラスメソッドとインスタンスメソッドの定義の仕方  
- クラスメソッドとインスタンスメソッドの呼び出し方  

について見ていきたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

#####シングルトンパターンのクラスの定義
まず、シングルトンパターンのクラスの定義の方法を見ていきましょう。  
Swift1.2までとSwift2.xでは定義の仕方が異なります。  
ここではSwift2.xの場合のみ書きます。  

```objective-c
class SingletonSample:NSObject {
	// シングルトンパターンとして定義
	static let sharedInstance:SingletonSample = SingletonSample()
	
	private init() {
	}
}
```

インスタンスは1つしか作られないので、  

- 内容に変更がない変数ということで`let`を使用  
- SingletonSampleクラスにおける固有の値ということで`static`を使用  
- 外部クラスからインスタンス化することを防ぐために`private`で`init`を定義  

となっています。  

シングルトンパターンのクラスを利用するときは、  
`SingletonSample.sharedInstance`でOKです。  
これで1つのインスタンスを使い回すことができます。  

#####インスタンスメソッドの定義
次にシングルトンパターンにおけるインスタンスメソッドの定義方法について見ていきましょう。  
まずは、定義する側です。  

```objective-c
class SingletonSample:NSObject {
	...
	// インスタンスメソッドの定義
	func sampleInstanceMethod() {
		...
	}
	...
}
```

インスタンスメソッドを使うときは  

```objective-c
SingletonSample.sharedInstance.sampleMethod()
```

という形で使うことができます。  
`SingletonSample.sharedInstance`で1つのインスタンスを使うことができるので、通常のクラスと使い方は同じですよね。  

#####クラスメソッドの定義
最後にシングルトンパターンにおけるクラスメソッドの定義方法について見ていきましょう。  
まずは、定義する側です。  

```objective-c
class SingletonSample:NSObject {
	...
	// クラスメソッドの定義
	class func sampleClassMethod() {
		...
	}
	...
}
```

クラスメソッドを使うときは  

```objective-c
SingletonSample.sampleClassMethod()
```

という形で使うことができます。  
クラスメソッドはクラスをインスタンス化する必要なく、利用できるメソッドなので、  
`SingletonSample.sharedInstance`とインスタンスを通して利用せずに、 **クラス名+メソッド名** で利用できるんですね。  

本日は簡単ですが、以上になります。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
