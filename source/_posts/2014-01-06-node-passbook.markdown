---
layout: post
title: "node-passbookではまったこと"
date: 2014-01-06 22:45
comments: true
categories: ios Passbook node.js
---

###PassbookのPassを動的生成するためにnode-passbookを使ってみました
皆様、あけましておめでとうございます。2014年1回目のブログ更新です。  
今回は**node-passbookではまったこと**について紹介したいと思います。通常、はまらないことかもしれないのですが、筆者は2日半くらい悩みました。  

node-passbookは[Github](https://github.com/assaf/node-passbook)上で公開されています。

<!--more-->

####node-passbookの設定
今回は、nodeのWebフレームワークであるExpressを使います。Expressの始め方については[gihyo.jsさんのページ](http://gihyo.jp/dev/serial/01/nodejs/0003)を参考にしてください。Expressの準備が整っている前提で説明を始めたいと思います。

まずはnode-passbookモジュールをインストールしましょう。  
```
npm install passbook
```

次に、app.jsを編集しましょう。
```
var express = require('express');
var routes = require('./routes');
var user = require('./routes/user');
// passbook発行処理を書いたpass.jsをrequire
var pass = require('./routes/pass');

<省略>

app.get('/', routes.index);
app.get('/users', user.list);
// passbookのpassをダウンロードするときのget
app.get('/download', pass.download);
// passbookのpassを作成するときのget
app.get('/create', pass.create);

<省略>

```

では、passbook発行処理を書いたpass.jsをroutes配下に作成します。
```
var fs = require('fs');
var path = require('path');
var createTemplate = require("passbook");

exports.create = function(req, res) {
	// Passbook作成
	var template = createTemplate('eventTicket', {
		'passTypeIdentifier': 'pass.****.****.****',
		'teamIdentifier': '***********',
		'organizationName': 'Passbookテスト'
	});
	template.keys('./public/keys', 'test12345');
	var passbook = template.createPass({
		serialNumber: 'abcdefc',
		description: 'Passbookテスト用のチケットです。',
		logoText: '参加チケット',
		foregroundColor: 'rgb(255, 255, 255)',
		backgroundColor: 'rgb(60, 65, 76)'
	});
	passbook.fields.webServiceURL = "https://example.com/passes";
	passbook.fields.authenticationToken = "vxwxd7J8AlNNFPS8k0a0FfUFtq0ewzFdc";
	passbook.fields.relevantDate = "2014-01-25T17:00+09:00";
	passbook.fields.barcode = {
		message: "http://google.com",
		format: "PKBarcodeFormatQR",
		messageEncoding: "iso-8859-1",
		altText: "ユーザID"
	};
	passbook.primaryFields.add([
		{
			"key" : "event",
			"label" : "イベント",
			"value" : "Passbookテスト"
		}
	]);
	passbook.secondaryFields.add([
		{
			"key" : "loc",
			"label" : "場所",
			"value" : "東京近辺"
		}
	]);
	passbook.backFields.add([
		{
			"key" : "message",
			"label" : "メッセージ",
			"value" : "Passbook盛り上げていきましょう！"
		}
	]);
	passbook.loadImagesFrom("./public/images");

	var pkpassFile = fs.createWriteStream("mypass.pkpass");
	passbook.on("error", function(error) {
		console.error(error);
		process.exit(1);
	});
	passbook.pipe(pkpassFile);
			
	res.end();
};

exports.download = function(req, res){

	var file = './mypass.pkpass',
		filestream = fs.createReadStream(file),
		filename = path.basename(file),
		mimetype = 'application/vnd.apple.pkpass';
	res.setHeader('Content-disposition', 'attachment; filename=' + filename);
	res.setHeader('Content-type', mimetype); 
	filestream.on('data', function(chunk) {
		res.write(chunk);
	});
	filestream.on('end', function() {
		res.end();
	});
};

```
さて、これでソースの準備は完了です。  
次の章では、筆者がはまった点について具体的に説明します。  

####証明書ではまりました
筆者は上記のソースを書き終わった時点で完了だと思ったのですが、iPhoneから落としてもうまくいきませんでした。  
うまくいかなかったときの現象は下記です。  
iOS6 ⇒ Pass自体は表示されるものの、追加ボタンを押しても追加されない  
iOS7 ⇒ Pass自体表示されない  

また、XcodeのOrganizerのコンソールでログを見たところ、**Signature is missing**と表示されてしまいました。  
悩んだ果てにnode-inspectorでデバッグして1つ1つ処理を確認したところ、**node_modules/passbook/lib/pass.js**の342〜350行目を見て気づきました。  

```
 function signManifest(template, manifest, callback) {
	 var identifier = template.passTypeIdentifier().replace(/^pass./, "");
	 
	 var args = [
		"smime",
		"-sign", "-binary",
		"-signer",    Path.resolve(template.keysPath, identifier + ".pem"),
		"-certfile",  Path.resolve(template.keysPath, "wwdr.pem"),
		"-passin",    "pass:" + template.password
	];

	<省略>
```

そう、pemファイルの名前が間違っていたのです。pemファイルの名前は**passTypeIdentifier**に設定する文字列から**pass.**を削除したものにしなくてはなりませんでした。  

やっぱりデバッグは大切ですね。

参考ページ:  
[node.jsでPassbookサーバをつくってみる](http://blog.techfirm.co.jp/2012/10/23/node-js%E3%81%A7passbook%E3%82%B5%E3%83%BC%E3%83%90%E3%82%92%E3%81%A4%E3%81%8F%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%8B1/)  
[Node.jsでPassbookを発行してみる](http://kataok.hatenablog.com/entry/2012/10/20/102125)  
