---
layout: post
title: "Swift ExpressでAjax通信してみよう！"
date: 2016-07-31 23:40
comments: true
categories: ios swift
---

###Swift ExpressでPOSTリクエスト投げてみよう！
さて、本日は[Swift Express](https://github.com/crossroadlabs/Express)の続きを試してみます。  
[前回](http://grandbig.github.io/blog/2016/07/10/swift-express/)はインストール方法とデフォルト画面の表示までを紹介しましたが、  
今回はAjax通信によるPOSTリクエストを投げてみたいと思います。  

####サーバサイドの対応
まずは、サーバサイドの準備です。  
`main.swift`を修正しましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

```objective-c
import Express

let app = express()

app.views.register(StencilViewEngine())
<!-- ここを追加 -->
app.views.register(JsonView())

app.get("/assets/:file+", action: StaticAction(path: "public", param:"file"))

app.get("/") { (request:Request<AnyContent>)->Action<AnyContent> in
    return Action<AnyContent>.render("index", context: ["hello": "Hello,", "swift": "Swift", "express": "Express!"])
}

<!-- ここを追加 -->
app.post("/:param") { (request:Request<AnyContent>)->Action<AnyContent> in
    let response = ["status": "ok", "description": "Post Request Succesfully"]
	    return Action.render(JsonView.name, context: response)
}

app.listen(9999).onSuccess { server in
    print("Express was successfully launched on port", server.port)
}

app.run()
```

`app.views.register(JsonView())`を追加することで、JSON形式のデータを返却できるようになります。  
また、返却レスポンスは`dictionary`型で書けばOKなようです。  

####クライアントサイドの対応
続いて、クライアントサイドの準備です。  
手っ取り早くjQuery使いましょう。  
[こちらのページ](https://jquery.com/)から最新のjQueryをダウンロードしてきます。  

そして、`index.js`をさくっと作っちゃいましょう。  

```javascript
$(function() {

	$("#touch").on("click", function() {
		// Ajax通信
		ajax();
	});

	function ajax() {
		$.ajax({
			type: "POST",
			url: "http://localhost:9999/1",
			dataType: "json"
		}).done(function(data) {
			alert(data.status + "\n" + data.description);
		}).fail(function(data) {
			alert("error!!");
		});
	}
});
```

これらjsファイルを読み込みます。  

```html
// index.stencil
<html>
	<head>
		<title>{{hello}} {{swift}} {{express}}</title>
		<link href='https://fonts.googleapis.com/css?family=Source+Sans+Pro:700italic,700' rel='stylesheet' type='text/css'>
		<link href='/assets/css/main.css' rel='stylesheet' type='text/css'>
	</head>
	<body>
		<img class="logo" src="/assets/logo.png"/>
		<h1>{{hello}} <i><span class="swift">{{swift}}</span> {{express}}</i></h1>
		<div id="touch">ここをタッチ！！</div>
															
		<script src="/assets/js/jquery-3.1.0.min.js"></script>
		<script src="/assets/js/index.js"></script>
	</body>
</html>
```

これだけで下図のようにPOSTリクエストが通りました。  

![POSTリクエスト](/images/swift-express-3.png)  

というところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
