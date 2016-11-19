---
layout: post
title: "Swift2.0で追加された新機能を見てみよう！"
date: 2016-02-29 21:38
comments: true
categories: ios swift
---

####今更だけど見てみたいSwift2.0で追加された新機能
さて、本日は表題通り、今更なんだけど...Swift2.0で追加された新機能について見ていきたいと思います。  
Swift2.0では下記3つの新機能が追加されました。  

* `guard statement`  
* `do-catch`, `try`, `defer`  
* `Protocol Extension`  

これらの使いどころやメリットなど見ていきたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####guard statementについて
まずは`guard statement`について見ていきます。  

#####用途について
条件分岐(`if 〜 else 〜`)で利用します。  
例えば、Open Weather Map APIでのリクエストでは  

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

のように書くことができます。  
以前であれば、下記のようにこれでもかってくらいネストしていたのですが、可読性が向上します。  

```objective-c
Alamofire.request(.GET, "http://api.openweathermap.org/data/2.5/weather?APPID=<あなた自身のAPPID>",
	parameters:["q":location]).responseJSON { (response) -> Void in
		if response.result.isSuccess {
			if let value = response.result.value {
				if let weatherArray = value["weather"] {
					if let weather = weatherArray?[0] {
						if let description = weather["description"] as? String {
							print(description)
						}
					}
				}
			}
		}
}
```

####do-catch, try, deferについて
続いて、`do-catch`, `try`, `defer`について見ていきます。  

#####用途について
基本的には、例外処理を実行したい場合に使います。  
想定外の例外が発生した場合にアプリが落ちないようにしないことはもちろんのこと、  
想定した例外処理が発生した場合にもそれ相応の処理を実施することがあるので、  
そういったときに使います。  

まずは、呼び出し元を書きます。  

```objective-c
// エラー種別の設定
enum MyError: ErrorType {
	case NilError
	case StringError
}

// サンプルメソッド
func sampleMethod(param: String?) throws -> String {
	defer {
		print("メソッドが呼ばれました！")
	}

	if param == nil {
		throw MyError.NilError
	} else if param == "Error" {
		throw MyError.StringError
	}

	return param
}
```

`throw`で例外を投げる際にエラー種別を渡す必要があるため、エラー種別を設定します。  
`defer`は例外発生の有無に関わらず、 **必ず実行する処理内容** を書きます。  

続いて、呼び出し先を書きます。  

```objective-c
do {
	let result = try self.sampleThrowMethod("Error")
	print(result)
} catch MyError.NilError {
	print("nilがパラメータとして渡されました")
} catch MyError.StringError {
	print("Error文字列がパラメータとして渡されました")
} catch {
	print("想定外のErrorが発生しました")
}
```

`do-catch`は例外発生別にその後の処理を分けるために利用します。  
`try!`で強制実行する場合は`do-catch`は不要ですが、  
アプリがダウンしては元も子もないのでオススメしません。  

####Protocol Extension
最後に`Protocol Extension`です。  

#####用途について
筆者のイメージではObjective-Cで言うところの **カテゴリ** の用途に近いのかなと思っています。  
Objective-Cのカテゴリと異なるのは指定した既存クラスにメソッドを追加するのではなく、  
**プロトコル** にメソッドを追加するので拡張性や柔軟性が高いようです。  

```objective-c
// MyProtocol.swift
protocol MyProtocol {
	func pico()
}

extension MyProtocol {
	func pico() {
		print("プロトコルメソッドを追加しました")
	}
}
```

定義したメソッドを下記のように呼び出せます。  

```objective-c
// ViewController.swift
class ViewController: UIViewController, MyProtocol {
	override func viewDidLoad() {
		super.viewDidLoad()

		self.pico()
	}
}
```

####まとめ
さて如何でしたでしょうか？  
今更ですが、Swift2.0の新機能について見てきました。  
個人的には`guard statement`はかなり利用したいなと思っています。  
ネストが多発しがちなSwiftでは活躍の場面が目に浮かびます。  
`do-catch`は待ち望んだ機能ですし、逆に今までなかったのが信じられないくらいですよね？  
より安定したSwift製アプリを作成することができると想像するとワクワクしますね。  

といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
