---
layout: post
title: "Server Side SwiftでMongoDBと遊んでみる"
date: 2016-12-05 00:00
comments: true
categories: swift mongodb BE
---

###はじめに
こちらは[Swift その2 Advent Calendar 2016](http://qiita.com/advent-calendar/2016/swift2) 5日目の記事です。  
今年は筆者が興味を持っている **Server Side Swift** について書きたいと思います。
Swiftサーバを立てるために、[以前の記事](http://grandbig.github.io/blog/2016/10/30/swift-perfect/)でも利用した[Perfect](https://github.com/PerfectlySoft/Perfect)を使います。  

ただ単にSwiftサーバを立てても面白くないので、提供されているMongoDB接続モジュールを利用して遊んでみようと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

###SwiftによるAP・DBサーバの構築
早速、Perfectを用いてSwiftによるAP・DBサーバを構築しようと思います。  
基本的には[ReadMe](https://github.com/PerfectlySoft/Perfect-MongoDB)に従えば良いのですが、丁寧に１つずつ見ていきます。  

####必要モジュールのインストール
流石に **Homebrew** はインストールされている方が多いと思いますが、入れていない方は下記コマンドで入れましょう。  

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

また今回は **MongoDB** を利用するため、 **mongodb** および **mongo-c** をインストールする必要があります。  
mongodbに関しては、  

```
brew install mongodb
```

でOKです。(自動起動などに関しては[こちら](http://grandbig.github.io/blog/2016/11/20/brew-install-db/)を参照ください。)  
続いて、mongo-cは

```
brew install mongo-c
```

でインストール完了できるはずです。  

####xcodeprojを作成
#####テンプレートのダウンロード
0からPerfectを用いてサーバを構築しても良いのですが、Perfectではテンプレートを用意してくれています。  
せっかくなので `git clone` してテンプレートをダウンロードして使いましょう。  

```
git clone https://github.com/PerfectlySoft/PerfectTemplate.git
cd PerfectTemplate
```

#####MongoDBモジュールを利用するように編集
**Perfect-MongoDB** を利用するので、`Package.swift`を編集しましょう。  

```objective-c
import PackageDescription

let package = Package(
	name: "PerfectTemplate",
	targets: [],
	dependencies: [
		.Package(url: "https://github.com/PerfectlySoft/Perfect-HTTPServer.git", majorVersion: 2, minor: 0),
		.Package(url:"https://github.com/PerfectlySoft/PerfectLib.git", majorVersion: 2, minor: 0),
		.Package(url:"https://github.com/PerfectlySoft/Perfect-MongoDB.git", majorVersion: 2, minor: 0)
    ]
)
```

ReadMeでは **Perfect-HTTPServer** は書かれていないのですが、これを使った方が便利なので、テンプレートに残したまま進めます。  

#####Packageからxcodeprojを作成
`Package.swift`の編集が終わったら、下記コマンドを実行して`xcodeproj`ファイルを作成しましょう。  

```
swift package generate-xcodeproj
```

そうすれば下図のような結果が得られるはずです。  

![フォルダ構成](/images/perfect-mongo_1.png)  

####MongoDBにデータを作成
サンプルを作成するためにもデータがなきゃ話になりませんよね？  
ということでデータを入れましょう。  

```
// MongoDBにアクセス
$ mongo

// DBとCollectionの作成
> use test;
> db.createCollection('testCollection');
{ "ok" : 1 }

// DBの確認
> show dbs;
local  0.000GB
test   0.000GB

// Collectionの確認
> show collections
testCollection

// データのインサート
> db.testCollection.insert({name: 'takahiro', age: 30, hobby: 'blog'});
WriteResult({ "nInserted" : 1 })
> db.testCollection.insert({name: 'ichiro', age: 43, hobby: 'baseball'});
WriteResult({ "nInserted" : 1 })

// データの確認
> db.testCollection.find()
{ "_id" : ObjectId("58391469a8589d99c931303c"), "name" : "takahiro", "age" : 30, "hobby" : "blog" }
{ "_id" : ObjectId("58392222a8589d99c931303d"), "name" : "ichiro", "age" : 43, "hobby" : "baseball" }
```

####MongoDBにアクセスして全データを取得
データの作成も完了したので、実際にGETリクエストでMongoDBからデータを取得する処理を書いてみましょう。  
(ReadMeではPerfect-HTTPServerを利用しない方法で書かれていたため本記事とは若干異なります。)  

#####テンプレートファイルの確認
まずは、初めから作成されている処理内容を確認します。  
説明は下記ソースコードにコメントを書いたので参照ください。  

```objective-c
// main.swift
import PerfectLib
import PerfectHTTP
import PerfectHTTPServer

// HTTPサーバの生成
let server = HTTPServer()

// リクエストに対するルーティングを設定
var routes = Routes()
routes.add(method: .get, uri: "/", handler: {
    request, response in
    // レスポンスヘッダーの設定
    response.setHeader(.contentType, value: "text/html")
    // レスポンスボディの設定
    response.appendBody(string: "<html><title>Hello, world!</title><body>Hello, world!</body></html>")
    // レスポンス完了処理
    response.completed()
	}
)

// サーバにルーティング設定を適用
server.addRoutes(routes)

// ポートを設定
server.serverPort = 8181

// ドキュメントルートのパスを設定
server.documentRoot = "./webroot"

// arguments.swiftで定義されているメソッド
// 更なるサーバ定義が必要な場合はここを見ましょう(SSLなど)
configureServer(server)

do {
    // HTTPサーバの起動
    try server.start()
} catch PerfectError.networkError(let err, let msg) {
    print("Network error thrown: \(err) \(msg)")
}
```

上記をXcodeをからRunさせた状態で `http://localhost:8181` にアクセスしてみましょう。  
Hello Worldの結果が得られるはずです。  

![Hello World](/images/perfect-mongo_2.png)  

#####MongoDB関連処理ファイルの作成
続いて、MongoDBへの接続・切断やデータ取得などの処理を作成していきます。  
これは別クラスの中に書いていきましょう。  

今回はMongoDB関連の処理をハンドリングするということで`mongoHandler.swift`というファイルを作成します。  
因みにただ作成しただけでは、XcodeがCompile対象として正しく認識してくれないので、自身で設定を変えましょう。  
下図のようにTARGETSから実行ファイルを選択して、Compile Sourcesとして`mongoHandler.swift`を追加してください。  

![Compile Sourcesに追加](/images/perfect-mongo_3.png)  

では実際のソースを見ていきましょう。  

```objective-c
// mongoHandler.swift

import PerfectLib
import PerfectHTTP
import MongoDB

class MongoHandler {

  var client: MongoClient?
  var database: MongoDatabase?
  var collection: MongoCollection?

  // MongoDBへの接続処理
  fileprivate func connect() {

    // コネクション確立
    client = try! MongoClient(uri: "mongodb://localhost")

    // testデータベースへの接続
    database = client?.getDatabase(name: "test")

    // testCollectionコレクションへの接続
    self.collection = database?.getCollection(name: "testCollection")
  }

  // MongoDBからの切断処理
  fileprivate func close() {
    collection?.close()
    database?.close()
    client?.close()
  }

  func searchAll(request: HTTPRequest, _ response: HTTPResponse) {
    // MongoDBへの接続
    connect()
    // 指定コレクションが見つからない場合は処理終了
    guard let collection = self.collection else {
      return
    }

    // データ全件取得のためBSONオブジェクトを初期化してクエリとして設定
    let fnd = collection.find(query: BSON())

    // データ格納用に配列を定義
    var arr = [String]()

    // 取得したデータを配列に格納する
    // fndはMongoCursor型であり、for文での繰り返し処理が可能
    for x in fnd! {
        arr.append(x.asString)
    }

    // JSONStringに変換
    let returning = "{\"data\":[\(arr.joined(separator: ","))]}"

    // レスポンスデータとして設定
    response.appendBody(string: returning)
    // レスポンス処理完了
    response.completed()

    // MongoDBから切断
    close()
  }
}
```

MongoDBへの接続・切断処理は共通処理となることは容易に想像できるため、切り出しました。  

#####main.swiftからmongoHandler.swiftを呼び出す
さて、`main.swift`から`mongoHandler.swift`を呼び出してみましょう。  

```objective-c
// main.swift
// MongoHandlerの初期化
let mongoHandler = MongoHandler()

// GETリクエストでMongoDBのデータ全取得
routes.add(method: .get, uri: "/mongo", handler: {
    request, response in
    mongoHandler.searchAll(request: request, response)
})
```

これで`http://localhost:8181`にアクセスすれば下図のような結果が得られるはずです。

![データ全件取得結果](/images/perfect-mongo_4.png)   

####MongoDBにアクセスして指定のクエリでデータを取得
全件取得の方法はわかったので、続いてクエリありの検索を実行してみましょう。  
先程述べた通り、下記のようなデータが格納されています。  

```
// データの確認
> db.testCollection.find()
{ "_id" : ObjectId("58391469a8589d99c931303c"), "name" : "takahiro", "age" : 30, "hobby" : "blog" }
{ "_id" : ObjectId("58392222a8589d99c931303d"), "name" : "ichiro", "age" : 43, "hobby" : "baseball" }
```

`name`、`age`、`hobby`を指定して検索するのはそんなに難しくないと思います。  

#####nameを指定してデータを取得
わかりやすいところからと言うことで`name`を指定してデータを取得してみます。  
まずは`mongoHandler.swift`にクエリ指定のメソッドを追加しましょう。  

```objective-c
// mongoHandler.swift
import PerfectLib
import PerfectHTTP
import MongoDB

class MongoHandler {

    var client: MongoClient?
    var database: MongoDatabase?
    var collection: MongoCollection?

		fileprivate func connect() {

        // open a connection
        client = try! MongoClient(uri: "mongodb://localhost")

        // set database, assuming "test" exists
        database = client?.getDatabase(name: "test")

        // define collection
        self.collection = database?.getCollection(name: "testCollection")
    }

    fileprivate func close() {
        collection?.close()
        database?.close()
        client?.close()
    }

		// searchAllメソッドは省略

		// =====ここから追加========================================================
		func search(query: BSON, request: HTTPRequest, _ response: HTTPResponse) {
			// MongoDBへの接続
			connect()

			// 指定コレクションが見つからない場合は処理終了
      guard let collection = self.collection else {
          return
      }

      // クエリを指定して検索
      let fnd = collection.find(query: query)

      // データ格納用に配列を定義
      var arr = [String]()

			// 取得したデータを配列に格納する
	    // fndはMongoCursor型であり、for文での繰り返し処理が可能
      for x in fnd! {
          arr.append(x.asString)
      }

      // JSONStringに変換
      let returning = "{\"data\":[\(arr.joined(separator: ","))]}"

      // レスポンスデータとして設定
      response.appendBody(string: returning)
			// レスポンス処理完了
      response.completed()

			// MongoDBから切断
      close()
    }
}
```

`searchAll`との違いは引数に`BSON`型の`query`を追加しているところです。  

続いて、`main.swift`にGETリクエストで`name`パラメータをキャッチできるように処理を追加していきましょう。  

```objective-c
// main.swift
import PerfectLib
import PerfectHTTP
import PerfectHTTPServer
import MongoDB					// ここを追加

// 省略

// nameパラメータを指定したGETリクエストのハンドリング
routes.add(method: .get, uri: "/mongo/name/{name}", handler: {
  request, response in
	// BSON型オブジェクトの初期化(クエリとして渡す)
  let bson = BSON()
  defer {
		// 処理完了時にBSONオブジェクトを削除
    bson.close()
  }
	// クエリとして渡すパラメータをセット
  bson.append(key: "name", string: request.urlVariables["name"]!)
	// クエリを指定して検索
  mongoHandler.search(query: bson, request: request, response)
})
```

これだけで準備万端です。  
`http://localhost:8181/name/takahiro`にアクセスすると下記結果が得られます。  

![nameパラメータを指定したGETリクエスト結果](/images/perfect-mongo_5.png)_  

#####ObjectIdを指定してデータを取得
続いて、少しクエリの書き方に迷うかもしれない`ObjectId`を指定したデータ検索をしてみましょう。  
`mongoHandler.swift`には特に変更がありません。  
`main.swift`のみ変更していきます。  

```objective-c
// main.swift
import PerfectLib
import PerfectHTTP
import PerfectHTTPServer
import MongoDB
import libmongoc				// ここを追加

// 省略

// ObjectIdパラメータを指定したGETリクエストのハンドリング
routes.add(method: .get, uri: "/mongo/oid/{oid}", handler: {
  request, response in
	// BSON型オブジェクトの初期化(クエリとして渡す)
  let bson = BSON()
  defer {
		// 処理完了時にBSONオブジェクトを削除
    bson.close()
  }
	// bson_oid_t型のオブジェクトの初期化
  var oid: bson_oid_t = bson_oid_t()
	// String型からbson_oid_t型に変換
  bson_oid_init_from_string(&oid, request.urlVariables["oid"])
	// クエリとして渡すパラメータをセット
  bson.append(key: "_id", oid: oid)
	// クエリを指定して検索
  mongoHandler.search(query: bson, request: request, response)
})
```

肝なのが、`import libmongoc`をしているということです。  
このライブラリの各種メソッドを利用することでServer Side Swiftから`ObjectId`を指定したデータ検索が可能になります。  

少し詳しく説明すると、  
クエリとして`ObjectId`を渡すためには`public func append(key k: String, oid: bson_oid_t) -> Bool`を利用する必要があります。  
しかし、このメソッドの第二引数をよく見ると、`bson_oid_t`型となっています。  

GETリクエストの時点で`bson_oid_t`型でパラメータを渡すわけにもいかないので、  サーバサイド側で変換する必要があります。  
そのために利用するメソッドが`void bson_oid_init_from_string (bson_oid_t *oid, const char *str);`です。  
筆者もSwiftで初めて利用したのですが、このメソッドは戻り値が`void`型のため何も返ってきません。  
が、第一引数に参照渡しとして`bson_oid_t`型オブジェクトを設定することで、メソッドの処理結果が`oid`に格納されます。  
これでめでたくクエリとして`ObjectId`が設定できるわけです。  

では、`http://localhost:8181/oid/58392222a8589d99c931303d`にアクセスしてみましょう。  

![ObjectIdパラメータを指定したGETリクエスト結果](/images/perfect-mongo_6.png)  

####MongoDBにデータを保存
検索に関してはざっと見てきたので、MongoDBへの保存処理も見ていきましょう。  

#####insertメソッドでデータを保存
保存には`MongoClient`の`insert`メソッドを利用します。  
(他にも`save`メソッドもあります。)  

まずは `mongoHandler.swift` へのメソッド追加からです。  

```objective-c
// mongoHandler.swift
func save(query: BSON, request: HTTPRequest, response: HTTPResponse) {
	// MongoDBへの接続
	connect()

	// 指定コレクションが見つからない場合は処理終了
  guard let collection = self.collection else {
    return
  }

	// データの保存処理
  let insert = collection.insert(document: query) as MongoResult
	// データ保存結果によって返却値を変更
  var returning = "Failed"
  switch insert {
  case .success:
    returning = "Succeeded"
  default:
    returning = "Failed"
  }

	// レスポンスデータとして設定
  response.appendBody(string: returning)
	// レスポンス処理完了
  response.completed()

	// MongoDBから切断
  close()
}
```

次にPOSTで届いたリクエストパラメータがJSONStringなのでDictionary型に変換します。  
その処理を `decode.swift` ファイルを新規作成して追加します。  

```objective-c
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

そして `main.swift` にPOSTリクエストのハンドリング処理を書きます。  

```objective-c
// main.swift
routes.add(method: .post, uri: "/mongo/", handler: {
	request, response in
	// BSON型オブジェクトの初期化(クエリとして渡す)
  let bson = BSON()
  defer {
		// 処理完了時にBSONオブジェクトを削除
    bson.close()
  }

	// JSONString型をDictionary型に変換
  let decodedParam = decode(postBody: request.postBodyString)
	// 各パラメータ単位でBSONオブジェクトに格納
  for (key, value) in decodedParam! {
      switch key {
      case "name":
          bson.append(key: "name", string: value as! String)
      case "age":
          bson.append(key: "age", int32: Int32(value as! Int))
      case "hobby":
          bson.append(key: "hobby", string: value as! String)
      default:
          break
      }
  }
	// データ保存
  mongoHandler.save(query: bson, request: request, response: response)
})
```

これで処理は完成です。  
ではPOSTリクエストを投げてみましょう。  

```
$ curl http://localhost:8181/mongo -X POST -H "Content-Type: application/json" -d '{"name":"Hanako", "age": 24, "hobby": "game"}'
```

再度、MongoDBを検索してみると下記のような結果が得られるはずです。  

```
// データの確認
> db.testCollection.find()
{ "_id" : ObjectId("58391469a8589d99c931303c"), "name" : "takahiro", "age" : 30, "hobby" : "blog" }
{ "_id" : ObjectId("58392222a8589d99c931303d"), "name" : "ichiro", "age" : 43, "hobby" : "baseball" }
{ "_id" : ObjectId("583afcaedcab265c1821fb51"), "name" : "Hanako", "age" : 24, "hobby" : "game" }
```

因みに、`main.swift`でリクエストパラメータとして取得した`age`を **Int32** 型に変換しているのには理由があります。  
これを仮に `bson.append(key: "age", int: value as! Int)` とした場合、MongoDBには `NumberLong`として保存されてしまいます。   
これは32bitか64bitかの違いですね。  

#####insertメソッドのMongoInsertFlagについて
先程、`insert`メソッドを利用しましたが、実は第一引数のみ持っている `insert` メソッドを利用していました。  
実は他にも `public func insert(document: BSON, flag: MongoInsertFlag = .none) -> Result` といった第二引数を持つ `insert` メソッドが存在します。  

少し気になったのでこの `MongoInsertFlag`について調べてみました。  
`MongoInsertFlag`は`MongoCollection.swift`内に **enum** として定義されています。  

```objective-c
public enum MongoInsertFlag: Int {
	case none
	case continueOnError
	case noValidate

	var mongoFlag: mongoc_insert_flags_t {
		switch self {
		case .none:
			return MONGOC_INSERT_NONE
		case .continueOnError:
			return MONGOC_INSERT_CONTINUE_ON_ERROR
		case .noValidate:
			return mongoc_insert_flags_t(rawValue: MONGOC_INSERT_NO_VALIDATE)
		}
	}
}
```

ここで`MONGOC_INSERT_NONE`, `MONGOC_INSERT_CONTINUE_ON_ERROR`, `MONGOC_INSERT_NO_VALIDATE`の3つがフラグとして用意されていることがわかります。  
これらはそれぞれ次のような意味とのことです。  

* `MONGOC_INSERT_NONE`
  * 特別何もしません。  
* `MONGOC_INSERT_CONTINUE_ON_ERROR`
  * 途中でエラーが発生したとしても後続の`insert`処理がある場合は続ける  
* `MONGOC_INSERT_NO_VALIDATE`  
  * インサート前に値のバリデーションチェックをしない  
    * MongoDBへの保存処理前にAPサーバ時点などでバリデーションチェックはした方が良い  
  * これをすることで処理時間を短縮することができる  

####MongoDBのデータを更新
MongoDBへの更新処理も見ていきましょう。  

#####updateメソッドでデータを保存
保存には`MongoClient`の`update`メソッドを利用します。  

`mongoHandler.swift`にメソッドを追加します。  

```objective-c
// mongoHandler.swift
func update(update: BSON, selector: BSON, request: HTTPRequest, response: HTTPResponse) {
	// MongoDBへの接続
	connect()

	// 指定コレクションが見つからない場合は処理終了
  guard let collection = self.collection else {
    return
  }

	// データの更新
  let updated = collection.update(update: update, selector: query)
	// データ更新結果によって返却値を変更
  var returning = "Failed"
  switch updated {
  case .success:
    returning = "Succeeded"
  default:
    returning = "Failed"
  }

	// レスポンスデータとして設定
  response.appendBody(string: returning)
	// レスポンス処理完了
  response.completed()

	// MongoDBから切断
  close()
}
```

そして `main.swift` にPUTリクエストのハンドリング処理を書きます。

```objective-c
// main.swift
routes.add(method: .post, uri: "/mongo/", handler: {
	request, response in
	// BSON型オブジェクトの初期化(更新内容として渡す)
  let updateBson = BSON()
	// BSON型オブジェクトの初期化(クエリとして渡す)
	let selectorBson = BSON()
  defer {
		// 処理完了時にBSONオブジェクトを削除
    updateBson.close()
		selectorBson.close()
  }

	// JSONString型をDictionary型に変換
  let decodedParam = decode(postBody: request.postBodyString)
	// 各パラメータ単位でBSONオブジェクトに格納
  for (key, value) in decodedParam! {
      switch key {
      case "name":
          updateBson.append(key: "name", string: value as! String)
      case "age":
          updateBson.append(key: "age", int32: Int32(value as! Int))
      case "hobby":
          updateBson.append(key: "hobby", string: value as! String)
      default:
          break
      }
  }
	// データ更新
  mongoHandler.update(update: updateBson, query: selectorBson, request: request, response: response)
})
```

基本的にはPOSTと同じ感じで処理を書くことができます。  
しかしながら気をつけなくてはいけないのが、**$setが利用できない** ということです。  
通常、MongoDBでは、 **$setを使わないと** ...  

```
// データの確認
> db.testCollection.find();
{ "_id" : ObjectId("583afcaedcab265c1821fb51"), "name" : "Hanako", "age" : 24, "hobby" : "game" }

// データの更新
> db.testCollection.update({_id: ObjectId("583afcaedcab265c1821fb51")}, {"hobby": "go shopping"});

// データの確認
> db.testCollection.find();
{ "_id" : ObjectId("583afcaedcab265c1821fb51"), "hobby" : "go shopping" }
```

となってしまいます。  
現状、`Perfect-MongoDB`に実装されているメソッドを見ると`$set`はないようです。  
そのため、今回はあえてPOST同様に全てのパラメータをクライアント側から下記のように投げることにしました。  

```
$ curl http://localhost:8181/mongo/583afcaedcab265c1821fb51 -X PUT -H "Content-Type: application/json" -d '{"name" : "Hanako", "age" : 24, "hobby": "game"}'
```

#####updateメソッドのMongoUpdateFlagについて
`update`メソッドにも実は`MongoUpdateFlag`というオプションを設定できるメソッドが存在します。  
`MongoUpdateFlag`は`MongoInsertFlag`と同じく **enum** として定義されています。  

```objective-c
public enum MongoUpdateFlag: Int {
	case none
	case upsert
	case multiUpdate
	case noValidate

	var mongoFlag: mongoc_update_flags_t {
		switch self {
		case .none:
			return MONGOC_UPDATE_NONE
		case .upsert:
			return MONGOC_UPDATE_UPSERT
		case .multiUpdate:
			return MONGOC_UPDATE_MULTI_UPDATE
		case .noValidate:
			return mongoc_update_flags_t(rawValue: MONGOC_UPDATE_NO_VALIDATE)
		}
	}
}
```

それぞれのフラグの意味は下記の通りです。  

* `MONGO_UPDATE_NONE`  
  * 特別何もしません  
* `MONGOC_UPDATE_UPSERT`  
  * 検索に引っかからない場合は`insert`します  
* `MONGOC_UPDATE_MULTI_UPDATE`  
  * 検索にヒットする件数が複数の場合は全て更新します  
* `MONGOC_UPDATE_NO_VALIDATE`  
  * アップデート前に値のバリデーションチェックをしません  

####MongoDBのデータを削除
MongoDBからのデータ削除処理も見ていきましょう。  

#####removeメソッドでデータを保存
保存には`MongoClient`の`remove`メソッドを利用します。  

`mongoHandler.swift`にメソッドを追加します。

```objective-c
// mongoHandler.swift
func delete(query: BSON, request: HTTPRequest, response: HTTPResponse) {
	// MongoDBへの接続
	connect()

	// 指定コレクションが見つからない場合は処理終了
  guard let collection = self.collection else {
    return
  }

	// データ削除処理
  let removed = collection.remove(selector: query, flag: .none)
	// データ削除結果によって返却値を変更
  var returning = "Failed"      
  switch removed {
  case .success:
    returning = "Succeeded"
  default:
    returning = "Failed"
  }

	// レスポンスデータとして設定
  response.appendBody(string: returning)
	// レスポンス処理完了
  response.completed()

	// MongoDBから切断
  close()
}
```

そして `main.swift` にDELETEリクエストのハンドリング処理を書きます。  

```objective-c
// main.swift
routes.add(method: .delete, uri: "/mongo/{oid}", handler: {
  request, response in
	// BSON型オブジェクトの初期化(クエリとして渡す)
  let bson = BSON()
  defer {
		// 処理完了時にBSONオブジェクトを削除
    bson.close()
  }

  var oid: bson_oid_t = bson_oid_t()
  bson_oid_init_from_string(&oid, request.urlVariables["oid"])
  bson.append(key: "_id", oid: oid)

	// データ削除
  mongoHandler.delete(query: bson, request: request, response: response)
})
```

下記のようなDELETEリクエストを投げてみればデータが削除されていることが確認できるはずです。  

```
$ curl http://localhost:8181/mongo/583afcaedcab265c1821fb51 -X DELETE -H "Content-Type: application/json"
```

#####removeメソッドのMongoRemoveFlagについて
`remove`メソッドにも実は`MongoRemoveFlag`というオプションを設定できるメソッドが存在します。  
`MongoRemoveFlag`は`MongoUpdateFlag`と同じく **enum** として定義されています。  

```objective-c
public enum MongoRemoveFlag: Int {
	case none
	case singleRemove

	var mongoFlag: mongoc_remove_flags_t {
		switch self {
		case .none:
			return MONGOC_REMOVE_NONE
		case .singleRemove:
			return MONGOC_REMOVE_SINGLE_REMOVE
		}
	}
}
```

フラグの意味は下記の通りです。  

* `MONGOC_REMOVE_NONE`  
  * 特に何のオプションもつけません。  
  * 検索にヒットしたデータは全て削除します。  
* `MONGOC_REMOVE_SINGLE_REMOVE`  
  * 初めに該当したデータ1件のみを削除します。  

###まとめ
以上で基本的なCRUDに対応したAPサーバとDBサーバをSwiftとMongoDBで構築することができました。  
これから何かサービスでも...と思っていたら時間切れ...  
今回は一旦ここまでとしたいと思いますが、今後の展望としてはSwift製iOSアプリと連携させてiOSアプリを作成するか、もしくはReact / Reduxを使ったWebサービスと連携させたいと企んでいます。  
処理速度とかリソースの消費具合とかは全然比較もしていないのでわからないですが、Swiftによるサーバサイド構築によるメリットも今後Appleさんが？明らかにしてくれるかもしれません。  

何と言っても新しい技術や取り組みは楽しいですね！！  
と言ったところで本日はここまで。  

###参考URL

* [Perfect MongoDB Documentation](http://perfect.org/docs/MongoDB.html)  
* [Libbson API Reference: Bson Oid Init](http://mongoc.org/libbson/1.3.5/bson_oid_init_from_string.html)  
* [Libbson API Reference: Update Document](http://mongoc.org/libmongoc/1.4.0/updating-document.html)  
* [MongoDB C Driver API Reference: Insert Flag](http://mongoc.org/libmongoc/1.0.0/mongoc_insert_flags_t.html)  
* [MongoDB C Driver API Reference: Update Flag](http://mongoc.org/libmongoc/1.2.3/mongoc_update_flags_t.html)  
* [MongoDB C Driver API Reference: Remove Flag](http://mongoc.org/libmongoc/1.3.2/mongoc_remove_flags_t.html)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
