---
layout: post
title: "SwiftでWebSocket!"
date: 2015-03-11 14:30
comments: true
categories: ios WebSocket socket.io swift
---

####socket.io公式クライアントソースを使ったサンプルの作成
先日、公開された[socket.io公式のクライアントソース](https://github.com/socketio/socket.io-client-swift)を使って、サンプルを作成したいと思います。  
公式にも[サンプルコード](https://github.com/nuclearace/socket.io-client-swift-example)は公開されているのですが、練習がてら試してみたいと思います。  

因みに本件にはサーバ側の説明は特に書いていません。  
下記、内容を試したい場合は[GitHub: socket.io.web](https://github.com/grandbig/socket.io.web)からソースをダウンロードしてお使いください。

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

#####プログラミング前の準備

まずは、[SocketRocket](https://github.com/square/SocketRocket)をダウンロードしてきましょう。  
READMEに書かれている通り、SocketRocketフォルダごと、自身のフォルダにコピーします。  

続いて、下記４つのFrameworkを追加しましょう。  

* libicucore.dylib
* CFNetwork.framework
* Security.framework
* Foundation.framework

本プロジェクトはSwiftがベースであるため、このままだと、Objective-Cのクラスを読み込めません。  
そこで、 **Bridge-Header** ファイルを作成する必要があります。  

作成手順は下記です。

1. プロジェクトを右クリックでAdd files to "プロジェクト名"を選択
2. TARGETS > Build Settings > Swift Compiler - Code Generation > Objective-C Bridging Header に作成したBridge-Headerファイルを指定
3. 作成したBridge-Headerファイルに利用したいObjective-Cファイルをimportする

そして、socket.io公式のクライアントソースである **SwiftIO** を追加しましょう。  

ここまでで準備完了です。  
下記のようなプロジェクト構成になっていることを確認してください。  

![プロジェクト構成](/images/ws_swift_1.png)  

#####プログラミング
さて、ここからは本格的にプログラミングを書いていきます。  
と言っても、今回作成するサンプルは[socket.io 公式HP > Get Started > Write a chat application](http://socket.io/get-started/chat/)を途中まで作成する程度なので、少ししか書きません。  

まずは、Storyboardにテキストフィールドとボタンを設置します。  

![Storyboardの様子](/images/ws_swift_2.png)  

続いて、StoryboardからViewController.swiftにリンク付けします。  
その結果、下記の通りになります。  
(ついでに後々必要となる部分も実装しています。)  

```objective-c
// ViewController.swift
import UIKit

class ViewController: UIViewController, UITextFieldDelegate {
	// Storyboardからのリンクづけ
	@IBOutlet weak var msgField: UITextField!

	override func viewDidLoad() {
		super.viewDidLoad()

		// Delegateの設定
		self.msgField.delegate = self
	}

	override func didReceiveMemoryWarning() {
		super.didReceiveMemoryWarning()
	}

	func textFieldShouldReturn(textField: UITextField) -> Bool {
		// キーボードのreturnを押下するとキーボードが閉じる処理
		self.msgField.resignFirstResponder()

		return true;
	}

	@IBAction func sendMessage(sender: AnyObject) {
		// Storyboardからのリンクづけ
		// 後ほど記載
	}
}
```

さて、これでほぼ大詰めです。  
WebSocket処理を書いてみましょう。


```objective-c
// ViewController.swift
import UIKit

class ViewController: UIViewController, UITextFieldDelegate {
	// Storyboardからのリンクづけ
	@IBOutlet weak var msgField: UITextField!

	// クライアント側のSocketコネクションの準備
	let socket = SocketIOClient(socketURL: "localhost:3000")

	override func viewDidLoad() {
		super.viewDidLoad()

		// Delegateの設定
		self.msgField.delegate = self

		// コネクションの接続
		self.socket.connection()
	}

	override func didReceiveMemoryWarning() {
		super.didReceiveMemoryWarning()
	}

	func textFieldShouldReturn(textField: UITextField) -> Bool {
		// キーボードのreturnを押下するとキーボードが閉じる処理
		self.msgField.resignFirstResponder()

		return true;
	}

	@IBAction func sendMessage(sender: AnyObject) {
		// Storyboardからのリンクづけ
		// テキストフィールド内の文字列をWebSocketで送る処理
		var msg = self.msgField.text
		self.socket.emit("chat message", msg)
	}
}
```

はい。これでクライアント側の実装終了です！  
上記のソースは全て[GitHub: socket.io.ios-swift](https://github.com/grandbig/socket.io.ios-swift)にアップしていますので、ご自由にご利用ください。  

ということで本日はここまで。  

参考:  

* [socket.io](http://socket.io/get-started/chat/)
* [socketio/socket.io-client-swift](https://github.com/socketio/socket.io-client-swift)
* [SocketRocketでWebSocketを実装してみる](http://qiita.com/n0bisuke/items/6dc990579b343475a635)
* [SwiftからObjective-Cのライブラリ使う](http://mk.hatenablog.com/entry/2014/10/16/075841)

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
