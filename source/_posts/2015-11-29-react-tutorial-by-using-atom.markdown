---
layout: post
title: "Atomエディタを使ってReact.jsを試してみた"
date: 2015-11-29 01:34
comments: true
categories: atom react.js
---

####Atomエディタとは
筆者はこれまでWeb系のファイルは全てvimで書いてきました。  
このまま、vimを使い続けても良いのですが、最近やっと一般的に認知され始めた [Atom](https://atom.io/) を使ってみようかなと思いつきで始めてみます笑  
(Atomが招待制だった頃に少々触ってはいたのですが、改めて触ってみると、便利ですね...。)  

因みにAtomとはGitHub製のオープンソースエディタです。  
2015年6月にバージョン1.0が公開され、本記事の執筆時点でバージョン1.2.4が最新版となっています。  
基本的な使いやすいと思った機能を下記にあげておきます。  

* HTML, CSS, JavaScriptにおける予測変換機能
* HTML, CSS, JavaScriptにおける対応する括弧の表示
* HTML, CSS, JavaScriptにおける属性の色彩表示

本日はAtomエディタを使って、基本的なReact.jsの使い方を勉強したいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####React.jsとは
さて、続いて、React.jsについて簡単に説明します。  
[React.js](https://facebook.github.io/react/index.html)とはFacebook製のUI構築ライブラリです。MVCで言うところの **View** のみをサポートします。MVC全てをサポートするわけではないため、[AngularJS](https://angularjs.org/)や[Backbone.js](http://backbonejs.org/)との併用が可能です。  

React.jsはその仕組から大規模設計の際にも管理が可能であるとのことなので、メリット/デメリットを見極めつつ、ケースバイケースで使うのが良いのでしょう。  
今回は勉強の意味を兼ねて、触ってみたいと思います。  

####React.jsを使って、基本的なことを書いてみる
本記事では **v0.14.0** のReact.jsを利用しています。  
〜v0.13.xまでと変更点もあるかと思うので、[Release Note](http://facebook.github.io/react/blog/2015/10/07/react-v0.14.html)は常に追うと良いでしょう。  

では、早速書いていきます。基本的には本家のチュートリアルに沿って学んでいきました。  
サンプルとして、クーポン一覧サイトを作ってみたいと思います。(実際のクーポンではありませんので、注意してください。)  

まず、HTMLファイルに関しては本家のGitHub上のソースをそのまま流用すると楽だと思います。  

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title>React Tutorial</title>
		<!-- Not present in the tutorial. Just for basic styling. -->
		<link rel="stylesheet" href="css/base.css" />
	</head>
	<body>
		<div id="content"></div>
		<script src="https://cdnjs.cloudflare.com/ajax/libs/react/0.14.0/react.js"></script>
		<script src="https://cdnjs.cloudflare.com/ajax/libs/react/0.14.0/react-dom.js"></script>
		<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.6.15/browser.js"></script>
		<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.1/jquery.min.js"></script>
		<script src="https://cdnjs.cloudflare.com/ajax/libs/marked/0.3.2/marked.min.js"></script>
		<script type="text/babel" src="js/coupon.js"></script>
		<script type="text/babel">
			// To get started with this tutorial running your own code, simply remove
			// the script tag loading scripts/example.js and start writing code here.
		</script>
	</body>
</html>
```

次に、サイトのレイアウトは下記とします。  

```
#content
┗━━ .couponBox
       ┗━━ .couponList
              ┗━━ .coupon
                     ┣━━ .couponImg
                     ┣━━ .couponName
                     ┗━━ .shopName
```

これに伴い、CSSファイルを簡単に下記としました。  

```css
#content {
  width: 96%;
}
.couponBox {
	  overflow: auto;
}
.couponList {
	  overflow: auto;
}
.coupon {
	  overflow: auto;
}
.couponImg {
	  width:200px;
	    float:left;
}
.couponInfo {
	  float: left;
}
```

そして、最も重要なJSファイルは下記です。  

```js
// テストデータ
var data = [
	{couponName: "ダブルチーズバーガー20円引き", shopName: "マ◯ドナルド新宿店", img: "http://urx.blue/pxQ3"},
	{couponName: "ビッグマック100円引き", shopName: "マ◯ドナルド新宿店", img: "http://urx.blue/pxQg"},
	{couponName: "フィレオフィッシュ30円引き", shopName: "マ◯ドナルド新宿店", img: "http://urx.blue/pxQl"}
];

// クーポン一覧をラップするDivタグの作成
var CouponBox = React.createClass({displayName: 'CouponBox',
	render: function() {
		return (
			<div className="couponBox">
				<CouponList data={this.props.data}/>
			</div>
		);
	}
});

// クーポン一覧Divタグの作成
var CouponList = React.createClass({
	render: function() {
		var couponNodes = this.props.data.map(function (coupon) {
			return (
				<Coupon couponName={coupon.couponName} shopName={coupon.shopName} img={coupon.img}></Coupon>
			);
		});
		return (
			<div className="couponList">
				{couponNodes}
			</div>
		);
	}
});

// クーポンDivタグの作成
var Coupon = React.createClass({
	render: function() {
		return (
			<div className="coupon">
				<img className="couponImg" src={this.props.img} />
				<div className="couponInfo">
					<div className="couponName">{this.props.couponName}</div>
					<div className="shopName">{this.props.shopName}</div>
				</div>
			</div>
		);
	}
});

// レンダリング
ReactDOM.render(
	<CouponBox data={data}/>,
	document.getElementById('content')
);
```

上記のソースコードより、下記の画面を作成しました。  
![サンプルのクーポンサイト](/images/atom_react_1.png)  

React.jsを使うことで、  

* レイアウトの要素をオブジェクト単位で捉えやすい
* レイアウトをツリー構造で捉えやすい

と感じました。  
まだチュートリアルの範囲でしか触れていないので、これから徐々に利用範囲を拡大していくことで学んでいきたいと思います。  
といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

