---
layout: post
title: "cache.manifestのキャッシュがかなり厄介!?"
date: 2013-10-04 00:35
comments: true
categories: html webview
---

###cache.manifestで苦労することまとめ
本日は筆者が日頃から困っているcache.manifestについて紹介したいと思います。  
『な〜んだWebか〜。ネイティブアプリとは関係ないかな〜』なんて思っていませんか？  
ネイティブアプリではWebViewを実装することができます。このWebViewにも大いに関わってくるところなので、知って損はないでしょう。  
 
<!--more-->

####cache.manifestの基本を簡単に紹介
cache.manifestを使ったことがあるという方は多いでしょうが、おさらい程度に基本を確認しておきましょう。  

```
# cache.manifestの記載例
CACHE MANIFEST

#version 1.0

CACHE:
image1.png

NETWORK:
base.css

FALLBACK:
```

・CACHEはキャッシュしたいファイルを記載します。  
・NETWORKは毎回サーバに問い合わせたいファイルを記載します。  
・FALLBACKはキャッシュできなかった場合の代わりのファイルを記載します。
・キャッシュしたいファイルの設定は変わらないがキャッシュの変更を読み込ませたいというときにversionをアップさせましょう。(なぜ必要か詳しくは例を交えて説明します)  


####cache.manifestを試そう
実際に簡単なhtml, css, cache.manifestを作成してcache.manifestの挙動を確認してみます。  

まずはcache.manifestを読み込む設定を書くindex.html
```
//index.html
<!DOCTYPE html>
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
次にcache.manifestの変更がわかりやすいようにcssを作成しましょう。
```
html, body {
	position: absolute;
	width: 100%;
	height: 100%;
	padding: 0;
	margin: 0;
	background-color: white;
}
#word {
	font-size: 30px;
	text-align: center;
}
```
これをChromeで表示すると
![Chromeで確認しよう](/images/cachetest1.png)  
このようになっています。  

画像はデバッガツールで詳細を見た様子です。cache.manifestで書いたimage1.pngだけでなくcache.manifestの読み込みを書いた**index.html自体もキャッシュ**されています。因みにbase.cssファイルはキャッシュされていないことも確認できます。  
base.cssは毎回読み込みに行くので変更内容は次回アクセス時に即座に反映されます。  
重要な事は**cache.manifestを編集していなくても反映される**ということです。  

一方でindex.htmlの内容を以下に変えてみたとします。
```
...
<body>
	<div id="word">Cache Test</div>
	<img src="./image1.png" />
	<div>test</div>
</body>
...

```
これでブラウザを更新してみましょう。divタグを追加したにも関わらず何も変化がないはずです。  
CACHEで設定したファイルの変更を知らせるためには必ずcache.manifestに変更を加えて下さい。(この場合はcache.manifestのバージョンを上げれば十分です。)
```
# cache.manifestのコメントアウトのバージョンを変更しよう
CACHE MANIFEST
 
#version 1.0.1
 
CACHE:
image1.png
 
NETWORK:
base.css
 
FALLBACK:
```
さて改めてブラウザを更新しましょう。あれ？まだ何も変わらない！と思うかもしれませんが、よくよくデバッガツールを見てみて下さい。  
![キャッシュが更新されている](/images/cachetest2.png)  
cache.manifestの更新を読み込んでいます。  

ここでもう一度ブラウザを更新しましょう。すると、先ほど追加したdivタグが表示されているはずです。  

つまり、cache.manifestの読み込みを書いたindex.html自体を更新するためには2回ブラウザを更新する必要があるということです。  
これをもし、iPhoneのネイティブアプリをWebViewを用いることで作るとしたらアプリの再起動が2回以上必要になります。ユーザにそれを強いるのはかなりハードルが高いと言えるでしょう。  

以上をまとめます。  
・cache.manifestの読み込みを設定したhtmlファイル(index.htmlなど)は更新することのないファイルとしましょう。  
・更新が生じる可能性のあるhtmlファイルは(index.htmlに)ロードするようにしましょう！  

本日はcache.manifestを用いた場合のキャッシュについて試しました。次回はネイティブ側でキャッシュ設定をした場合、どんな挙動を示すのか追いたいと思います。(cache.manifestと併用させた場合どうなるかも追えればと思います。)
