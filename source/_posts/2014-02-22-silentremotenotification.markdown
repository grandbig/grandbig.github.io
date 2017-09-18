---
layout: post
title: "Silent Remote Notificationを試そう！"
date: 2014-02-22 00:06
comments: true
categories: ios notification node
---

###iOS7で新たに追加されたSilent Remote Notificationを使ってみよう！！
本日はiOS7から新たに追加された**Silent Remote Notification**について紹介します。  
iOS6までで利用できたRemote Notificationと何が違うのかと言いますと、**ユーザ端末のロック画面に表示させることなく**通知することができるんです。  
つまり、  
Remote Notification: ユーザに何かを気づかせるためにプッシュする  
Silent Remote Notification: ユーザに気づかせることなくプッシュする  
ということです。  

これを見ると、『Silent Remote Notificationって使い道あるの？？』なんて思ってしまいそうですよね。これが意外とありそうなんです。  
例えば、  
・ユーザに気づかれずにアプリ内の情報を最新に更新する  
・ユーザに気づかれずにアプリ内のユーザ情報を取得する  
ことなどが考えられます。  
後者はユーザ視点からすると気になるかもしれませんが、『アプリの有用性をユーザに最大限に感じてもらうための方法』と考えて頂ければとても大切なことであると伝わるでしょうか。  

そんな様々な使い道がありそうなSilent Remote Notificationについて早速説明していきます。  

<!--more-->

####ネイティブソースとXcode5の設定について
まずは、ネイティブソースとXcode5の設定から説明します。Xcode5でプロジェクトを作成したら、TARGETS > Capabilities > Background Modes > Remote notificationsにチェックを入れます。  
![Xcode5での設定方法](/images/xcode-silent-notifications1.png)  
続いてはネイティブソースでプッシュの受取り口を作りましょう。  

```c
// AppDelegate.hファイル
@interfase AppDelegate : NSObject<UIApplicationDelegate> {
	NSString *token;
}
```
```c
// AppDelegate.mファイル
// ①トークン値の取得
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)devToken
{
	//デバイストークンから'<','>',' 'を削除して変数に保存
	self.token = [[[[devToken description] stringByReplacingOccurrencesOfString:@"<"withString:@""]
							stringByReplacingOccurrencesOfString:@">" withString:@""]
							stringByReplacingOccurrencesOfString: @" " withString: @""];

	NSLog(@"deviceToken: %@", self.token);
}

// ②通常のRemote Notificationが届いたときに通る処理
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo
{
	...
}

// ③Silent Remote Notificationが届いたときに通る処理
- (void)application:(UIApplication *)application
 didReceiveRemoteNotification:(NSDictionary *)userInfo
  fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
{
	...
}
```
プッシュ通知を受け取ったとき、上記に書いたdidReceiveRemoteNotification処理が実行されます。コメントアウトにも書いていますが、③がSilent Remote Notificationが届いたときに通る処理です。  
ただし、Silent Remote NotificationはiOS7以上でないと対応していないのでiOS6の場合は通常のRemote Notification, Silent Remote Notificationに関わらず②のdidReceiveRemoteNotificationを通りますので、ご注意ください。  
あとは③に実行させたい処理を書けば、Silent Remote Notificationの実装は完了です。  

####動作確認用に簡単にプッシュ通知を送る処理を書こう
さて、ネイティブ側の実装はできましたが、本当にこれで送れるのか確認したいですよね？Node.jsのnode-apnモジュールを使えば驚くほど簡単に用意することができます。  
より簡単に作成するためにExpressモジュールを使って説明します。Expressモジュールの用意については[第3回 Express.jsを使ったWebアプリケーションを構築](https://gihyo.jp/dev/serial/01/nodejs/0003)を参照ください。  

それではpushtestという名前のWebアプリの雛形を作成しましょう。  
```bash
express -t ejs pushtest
```
次にpushtest配下で  
```bash
npm install
```
を実行して必要なモジュールをインストールしましょう。さらに、node-apnモジュールもインストールしましょう。また後ほど利用するquerystringモジュールもインストールしておきましょう。  
```bash
npm install apn
npm install qs
```
これでモジュールが揃ったので、app.jsを編集しましょう。  
pushtest/routes/apns.jsを作成しておきます。(デフォルトで作成されるindex.jsかuser.jsをコピーしてファイル名を変更すればOKです)
```js
var express = require('express');
var routes = require('./routes');
var user = require('./routes/user');
var apns = require('./routes/apns');// apns.jsの読み込みを追加
var http = require('http');
var path = require('path');

var app = express();
...
app.get('/apns', apns.push);// GETリクエストを想定
...
```

そしてお待ちかねのプッシュメイン処理をapns.jsに書いていきます。  
```js
var apns = require('apn');
var qs = require('qs');

exports.push = function(req, res){
	// ① GETパラメータからトークンを取得
	var arr = req.url.split("?");
	var obj = qs.parse(arr[1]);
	var token = obj.token;

	// ② プッシュ通知の設定
	var options = {
		"cert": "./keys/cert.pem",
		"key": "./keys/key.pem",
		"gateway": "gateway.sandbox.push.apple.com"
	};
	var myDevice = apns.Device(token);
	var apnConnection = new apns.Connection(options);

	var note = new apns.Notification();
	note.contentAvailable = 1;

	apnConnection.pushNotification(note, myDevice);

	res.send("respond with a resource");
}
```
先ほどインストールしたqsはGETパラメータをオブジェクト化してくれる便りなモジュールです。詳しくは[Node.js Manual&Documents](http://nodejs.org/api/querystring.html)を参照してください。  
まず、①のようにトークン値を取得します。  
次に②のようにプッシュ通知を送るための設定を書きます。[GitHubで公開されているReference](https://github.com/argon/node-apn)をほぼそのまま書くだけなので簡単です。  
注意しておきたいのはSilent Remote Notificationの場合は**note.contentAvailable=1(note.newsstandAvailable=1でも可)**を設定する必要があります。逆にbadge, alert, soundは不要です。これらを入れてもプッシュ通知ができないわけではないのですが、冒頭に述べた**ユーザに気づかれずに**という要件を満たせなくなります。

因みに, optionsで設定しているcertとkeyは証明書の保存場所です。上記ソースで設定している場所はapp.jsと同じ階層にkeyフォルダがあることを指します。また、gatewayにはsandboxをつけることを忘れずに。(商用の場合はsandboxは不要です。develop用ではsandboxを設定するんです。)  
証明書(cert.pemとkey.pem)の作成は多くのサイトで説明されているのでここでは割愛します。  

ここまで準備ができたら、  
```bash
node app.js
```
でWeb&APサーバをローカルに起動しましょう。  
そして、『localhost:3000?token=トークン値』をブラウザのアドレス部分に打って、Silent Remote Notificationを送りましょう!!  
トークン値はネイティブソース内のdidRegisterForRemoteNotificationsWithDeviceToken関数でLogとして出力しているはずなので、コピーしましょう。  

届きましたでしょうか？
