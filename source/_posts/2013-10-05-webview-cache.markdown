---
layout: post
title: "WebViewのキャッシュの仕組みを見極めよう！"
date: 2013-10-05 02:47
comments: true
categories: webview
---

###WebViewとcache.manifestの関係性を知ろう！
前回はPCで見た場合のキャッシュについて調べました。本日も引き続きキャッシュについて調べてみたいと思います。  
筆者が最も疑問を感じていたのはiPhoneアプリ内でWebViewを利用したときキャッシュはどうなるのか？ということです。  
今日は実際に実機でテストをしながら進めます。  
そしてどうせならばiOS7搭載の実機で試してみましょう！  

<!--more-->

####cache.manifestのキャッシュはどこに保存される？
前回説明したcache.manifestを使用すると、そのキャッシュはどこに保存されるのか確認します。  
まずはcache.manifestをindex.htmlで読み込まない以下の場合で見てみましょう。
```
<!--index.html-->
<!DOCTYPE html>

<!--cache.manifestは読み込まない-->
<html>
	<head>
		<meta charset="UTF-8" content="text/css" http-equiv="Content-Style-Type">
		<meta content="width=device-width, user-scalable=no, minimum-scale=0.5, maximum-scale=1.0" name="viewport">
		<meta name="format-detection" content="telephone=no">
		<link href="./base.css" media="all" rel="stylesheet" type="text/css">
		<title>Cache-Test</title>
	</head>
	<body>
		<div id="word">Cache Test</div>
		<img src="./image1.png" />
	</body>
</html>
```

このページを簡単なWebViewアプリに表示します。  
CachePracticeという名前のアプリを作成してiOS7の端末にインストールしました。  
![iOS7にWebViewアプリをインストール](/images/cachepractice_1.PNG)  

さて立ち上げてみましょう！  
![cache.manifestを設定していないページをWebViewに表示](/images/cachepractice_2.PNG)  

このときのアプリ内のデータをPCにダウンロードして見てみましょう。 
![cache.manifestを設定していないページを読み込んだ場合](/images/cachepractice3.jpg)  
Cache.dbというものがありますね。  

では次に以下のcache.manifestを読み込ませたindex.htmlをWebViewで表示した場合を見てみましょう。
```
<!--index.html-->
<!DOCTYPE html>

<!--cache.manifestを読み込む-->
<html manifest="cache.manifest">
	<head>
		<meta charset="UTF-8" content="text/css" http-equiv="Content-Style-Type">
		<meta content="width=device-width, user-scalable=no, minimum-scale=0.5, maximum-scale=1.0" name="viewport">
		<meta name="format-detection" content="telephone=no">
		<link href="./base.css" media="all" rel="stylesheet" type="text/css">
		<title>Cache-Test</title>
	</head>
	<body>
		<div id="word">Cache Test</div>
		<img src="./image1.png" />
	</body>
</html>
```
![cache.manifestを設定したページをWebViewに表示](/images/cachepractice4.png)  
このようにApplicationCache.dbというものが作られました。どうやらcache.manifestでのキャッシュはここに保存されているようですね。  
それを裏付けるようにcache.manifestに画像を新たに追加したところApplicationCache.dbのサイズが増えました

####今後の調査について
cache.manifestについては大体わかりました。  
次に調べるとしたらネイティブ側のキャッシュ機構です。様々なサイトでネイティブキャッシュはCache.dbで保存されると書かれています。  
因みにSQLite 3.7.0以上から**WAL(Write-Ahead log)**方式になりました。現在普及しているiOSではSQLite3.7.0以上を利用しているため、自動的にWAL方式でキャッシュしています。
先ほど上げた画像を見てもCache.db, Cache.db-shm, Cache.db-walが生成されていることが確認できます。  

これら3つのファイルのサイズを眺めているとCache.db-walがどんどん増大していきます。一方でCache.dbのサイズは大きくならないんですよね...。このことからネイティブキャシュはDB(Cache.db)からキャッシュ情報を取り出すのではなく、ログファイル(Cache.db-wal)からキャッシュ情報をリカバリしているように思えます。
もっと詳しいことがわかれば追記したいと思います。  

とりあえず、今日はここまで。
