---
layout: post
title: "Server Side Swift: Perfect を使ってみよう！"
date: 2016-10-30 22:48
comments: true
categories: ios node swift BE
---

###Server Side SwiftライブラリのPerfect
本日は以前書いたサーバサイドSwiftの続きを書きます！  
と言いたかったところなのですが、Swift ExpressはSwift3.0やXcode8に対応しておらず、何もできなかったため、方向転換して最もSTAR数の多い[PerfectlySoft/Perfect](https://github.com/PerfectlySoft/Perfect)を使うことにしました。  

よくよく見るとMySQLだけでなくMongoDB接続用にもモジュールが用意されており、なかなか良さそうではないですか！！  
とは言いつつも、そんなにすぐにMaster Of Perfectにはなれないので少しずつ見ていくことにします。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
<!-- more -->

####チュートリアルを触ってみる
まずは何はともあれGitHubの *Getting Started* からやらないと話になりません。  
手順は簡単です。  

１．テンプレートプロジェクトをクローンする  
`git clone https://github.com/PerfectlySoft/PerfectTemplate.git`

２．ビルドを実行する  
クローンした`PerfectTemplate`フォルダ内に入り、ビルドを実行しましょう。  

```
$ cd PerfectTemplate
$ swift build
```

３．サーバを起動します  
なんと後は下記コマンドでサーバを起動するだけです。  

`.build/debug/PerfectTemplate`  

正しく起動すれば、下記ログが出力されます。  
`Starting HTTP server on 0.0.0.0:8181 with document root ./webroot`  
またログの指示通りChromeで`http://localhost:8181/`にアクセスすれば`Hello World`が拝めます。  

####ルーティングの書き方について学ぶ
では次に簡単なルーティングについて学んでいきましょう。  
チュートリアルでは、下記GETリクエストのみ受け付けていました。  

```javascript
// main.swift
var routes = Routes()
routes.add(method: .get, uri: "/", handler: {
	request, response in
	response.setHeader(.contentType, value: "text/html")
	response.appendBody(string: "<html><title>Hello, world!</title><body>Hello, world!</body></html>")
	response.completed()
})
```

最も単純なGETリクエストですね。  
では、`ID: 100`のユーザ情報を取得するGETリクエストはどう受け付けるのでしょうか。  

```javascript
// main.swift
var routes = Routes()
routes.add(method: .get, uri: "/user/{id}", handler: {
	request, response in
	response.setHeader(.contentType, value: "text/html")
	response.appendBody(string: "<html><title>Sample</title><body>You GET UserInfo with \(request.urlVariables)</body></html>")
	response.completed()
})
```

たったのこれだけです。  
では、POSTリクエストの受け付けはどうでしょうか。  

```javascript
// main.swift
var routes = Routes()
routes.add(method: .post, uri: "/user", handler: {
	request, response in
	response.appendBody(string: "<html><title>Sample</title><body>You POSTed user data to catch your POST request.</body></html>")
	response.completed()
})
```

これも簡単ですね。  
書き方に若干の違いはあれど、最早Node.jsとそんなに変わらん...  

おまけで、POSTリクエストで届いたJSONStringをバラバラっと分解して返却してみました。  
そのためにまずはJSONStringをデコードする処理を実装します。  

```javascript
// decode.swift
import PerfectHTTPServer
import PerfectLib

func decode(postBody: String?) -> [String: Any]? {
	do {
		guard let decoded = try postBody?.jsonDecode() as? [String:Any] else {
			return [:]
		}
		print(decoded)
		return decoded
	} catch {
		return [:]
	}
}
```

これを下記のように利用します。  

```javascript
// main.swift
var routes = Routes()
routes.add(method: .post, uri: "/user", handler: {
	request, response in

	var userInfo = ""
	let decodedParam = decode(postBody: request.postBodyString)
	for (key, value) in decodedParam! {
		switch key {
			case "name":
				userInfo = userInfo + "name is \(value as! String).\n"
			case "email":
				userInfo = userInfo + "email is \(value as! String)."
			default:
				break
		}
	}
	response.appendBody(string: "<html><body>POST handler: \(userInfo)</body></html>")
	response.completed()
})
```

さて、実装できたのでクライアントからリクエストを投げてみます。  

```
// クライアントからPOSTリクエストを投げます
curl http://localhost:8181/user -X POST -H "Content-Type: application/json" -d '{"name":"Ichiro", "email": "xxx@gmail.com"}'
// 結果
<html><body>POST handler: name is Ichiro.
email is xxx@gmail.com.</body></html>xxxx:PerfectTemplate
```

###まとめ
さて今回は`Perfect`を使ったサーバサイドSwiftを見てみました。  
まだまだ基本的なリクエストの受付しかみていませんが、既にいろいろなモジュールが用意されているようなので、継続的に見ていきたいと思います。  
やっぱりライブラリを作るなら、最新の状況についていかないと見捨てられるな〜と思ってしまいました。  
(今回で言うと、Swift3やXcode8とかですね。)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
