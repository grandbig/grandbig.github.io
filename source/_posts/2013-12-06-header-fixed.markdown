---
layout: post
title: "UIWebView内でheaderを固定して滑らかなスクロールを実現しよう！"
date: 2013-12-06 23:24
comments: true
categories: ios webview html
---

###header固定をしたスクロールは案外難しい！！
本日はUIWebViewを全面に貼り付けた上で、ヘッダーを固定して滑らかなスクロールを実現する方法について書きたいと思います。  
もし、ヘッダーをネイティブで作成して、中身をWebで作成すれば簡単に実現できます。しかし、ネイティブとWebのハイブリットアプリを目指すとAndroidへの対応が手間となるため一長一短な面があります。今回はあえて全面UIWebViewに拘りたいと思います。  

<!--more-->

####position: fixedだとうまくいかない！？
まず、Webでヘッダーを固定することを考えたときに真っ先に思い浮かぶものは**position: fixed**を利用することだと思います。  

サンプルHTML
```
<!DOCTYPE html>
	<head>
		<meta charset="UTF-8" content="text/css" http-equiv="Content-Style-Type" />
		<meta content="width=device-width, user-scalable=no,minimum-scale=0.5, maximum-scale=1.0" name="viewport" />
		<link href="./main.css" media="all" rel="stylesheet" type="text/css" />
		<title>サンプル</title>
	</head>
	<body>
		<div id="header">ヘッダー</div>
		<div id="mainContents">
			<div id="main">
				contents
				<br>
				contents
				<br>
				...
				contents
				<br>
			</div>
		</div>
	</body>
</html>
```

サンプルCSS
```
html, body {
	padding: 0;
	margin: 0;
	width: 100%;
	height: 100%;
}

#header {
	position: fixed;
	top: 0;
	left: 0;
	width: 100%;
	height: 64px;
	z-index: 2;
}

#mainContents {
	position: relative;
	top: 64px;
	width: 100%;
	height: 100%;
	-webkit-overflow-scrolling: touch;
	z-index: 1;
}
```
確かにこの設定であればヘッダーを固定して中身を滑らかにスクロールすることが可能になります。そして、iPhoneのSafariからその画面を見るのであればさして違和感はないでしょう。しかし、これをUIWebView上で見ると画面のバウンドを止められないことに気がつきます。これは結構な違和感です。  
このバウンドを以下のようにネイティブソース側から止めることを考えました。

ネイティブソース(UIWebViewの設定部分)
```
- (void)viewDidLoad {
	[super viewDidLoad];
	uiWebView.delegate = self;
	// バウンドしないように設定
	[[(UIScrollView *)[uiWebView subviews] objectAtIndex:0] setBounces:NO];
	...
}
```
しかし、これだとCSSで設定した**-webkit-overflow-scrolling: touch**の効果がなくなってしまいます。なので別の方法を考えました。

####position: absoluteを使う！！
position: fixed;は使いません。position: absolute;を使います。以下、踏まえて書き直します。  

サンプルHTML
```
<!DOCTYPE html>
	<head>
		<meta charset="UTF-8" content="text/css" http-equiv="Content-Style-Type" />
		<meta content="width=device-width, user-scalable=no,minimum-scale=0.5, maximum-scale=1.0" name="viewport" />
		<link href="./main.css" media="all" rel="stylesheet" type="text/css" />
		<title>サンプル</title>
	</head>
	<body>
		<div id="wrapper">
			<div id="header">ヘッダー</div>
			<div id="mainContents">
				<div id="main">
					contents
					<br>
					contents
					<br>
					...
					contents
					<br>
				</div>
			</div>
		</div>
	</body>
</html>
```

サンプルCSS
```
html, body {
	padding: 0;
	margin: 0;
	width: 100%;
	height: 100%;
	overflow: hidden;
}

#wrapper {
	position: relative;
	width: 100%;
	height: 100%;
}

#header {
	position: absolute;
	top: 0;
	left: 0;
	width: 100%;
	height: 64px;
	z-index: 2;
}

#mainContents {
	position: relative;
	top: 64px;
	width: 100%;
	height: 100%;
	overflow: auto;
	-webkit-overflow-scrolling: touch;
	z-index: 1;
}

#main {
	position: relative;
	padding: 64px 0;
}
```
これでほとんどの場合、バウンドしなくなります。しかも、滑らかなスクロールを保つこともできます。  
ただし、ヘッダーをタッチムーブすると...バウンドしてしまいます！！  

う〜ん。これは果たして無理なのか！？と思ったところで思いつきました。**touchイベントを無効化**しましょう！！  
jQueryを使っている前提で下記を紹介します。

サンプルJS
```

jQuery( function($) {
	
	$("#header").bind("touchstart", function() {
		event.preventDefault();
	});
	
	$("#header").bind("touchmove", function() {
		event.preventDefault();
	});

	$("#header").bind("touchend", function() {
		event.preventDefault();
	});

});
```
これでバウンドせずに滑らかなスクロールを保つことができました。  

と思ったのですが、画面の両端を触っているとバウンドが発生してしまいます...。要調査ですね...。
