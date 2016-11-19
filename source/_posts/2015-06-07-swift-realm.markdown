---
layout: post
title: "SwiftでRealmを使ってみよう！"
date: 2015-06-07 16:35
comments: true
categories: ios swift
---

####Realmをプロジェクトに追加しよう
本日は最近、流行っているRealmについて遊んでみたいと思います。  
まずは導入の仕方から見て行きましょう！  

#####Realmの導入方法
下記のPodfileを作成しましょう。  
本記事のSwiftプロジェクトとして、`SwiftRealm.xcodeproj`を作成します。  

```objective-c
// Podfile
platform :ios, '8.0'
source 'https://github.com/CocoaPods/Specs.git'

target 'SwiftRealm', exclusive: true do
	pod 'Realm'
end

target 'SwiftRealmTests', exclusive: true do
	pod 'Realm/Headers'
end
```

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

あとは`pod install`しましょう。  
すると、`SwiftRealm.xcworkspace`が作成されるので、Xcodeで起動しましょう。  

![Realmを追加した結果](/images/swift-realm.png)  

なんと、Objective-C++で書かれていますね！
Swiftで利用するためにはBridging-Headerファイルの設定を行う必要があります。  

SwiftRealm-Bridging-Header.hを作成します。  
![ヘッダーファイルの作成](/images/swift-realm2.png)  

TARGETS > SwiftRealm > Build Settings > Swift Compiler - Code Generation > Objective-C Bridging Header にパスを設定します。  
![パスの設定](/images/swift-realm3.png)  

SwiftRealm-Bridging-Header.hの中身を書きます。  

```objective-c
#ifndef SwiftRealm_SwiftRealm_Bridging_Header_h
#define SwiftRealm_SwiftRealm_Bridging_Header_h

#import <Realm/Realm.h>

#endif

```

これでRealmを利用する準備が整いました。  
実際にコードを書いていきましょう。  

#####保存オブジェクトの生成
Realmでは保存する対象を`RLMObject`として作成します。  
SQLiteでいうところのテーブルを生成しているイメージでしょうか。  

今回は練習として`Engineer.swift`内で`Engineer`クラスを作ってみました。  

```objective-c
// Engineer.swift

// Engineerクラス
class Engineer:RLMObject {
	// プロパティと初期値の設定
	dynamic var id:Int = 0
	dynamic var level:Int = 0
	dynamic var businessTitle:String = ""
	dynamic var skills = RLMArray(objectClassName: "Skills")
	dynamic var created:Double = 0
	dynamic var updated:Double = 0

	// プライマリーキーの設定
	override class func primaryKey() -> String? {
		return "id"
	}

	// インデックスの設定
	override class func indexedProperties() -> [AnyObject]? {
		return ["level"]
	}
}

// Skillsクラス
class Skills: RLMObject {
		dynamic var skill:String = ""
}
```

ポイントとしては、`skills`のように配列の形で1つの要素に複数の値を割り当てたい場合は、それ単体で別に`RLMObject`の拡張クラスを作成する必要があります。  
`RLMArray`を初期化することは許可されていないためです。  

#####オブジェクトのインサート/アップデート
続いて、先ほど生成したオブジェクトをインサート/アップデートしてみましょう。  

```objective-c
// ViewController.swift
import UIKit

class ViewController: UIViewController {
	
	override func viewDidLoad() {
		super.viewDidLoad()

		// id=0のオブジェクトを検索
		if let engineer = Engineer(forPrimaryKey: 0) {
			// 検索結果がある場合
			println("既にエンジニアが１人います。\(engineer)")

			// アップデート
			let realm = RLMRealm.defaultRealm()
			// 書込みの開始
			realm.beginWriteTransaction()
			// Skillsクラスをインスタンス化
			let skills = Skills()
			// 要素を設定
			skills.skill = "Objective-C"
			// 既存のデータに要素を追加
			engineer.skills.addObject(skills)
			engineer.updated = NSDate().timeIntervalSince1970
			// データのアップデート処理
			realm.addOrUpdateObject(engineer)
			// 書込みの終了
			realm.commitWriteTransaction()
		} else {
			// 検索結果がない場合
			// Engineerクラスをインスタンス化
			let newEngineer = Engineer()
			newEngineer.id = 0
			newEngineer.level = 0
			newEngineer.businessTitle = "新米エンジニア"
			// Skillsクラスをインスタンス化
			let skills = Skills()
			skills.skill = "Swift"
			newEngineer.skills.addObject(skills)
			newEngineer.created = NSDate().timeIntervalSince1970

			// データの新規作成
			let realm = RLMRealm.defaultRealm()
			// 書込みの開始
			realm.beginWriteTransaction()
			// データのインサート処理
			realm.addObject(newEngineer)
			// 書込みの終了
			realm.commitWriteTransaction()
		}
	}

	override func didReceiveMemoryWarning() {
		super.didReceiveMemoryWarning()
	}
}
```

ここでは、`id=0`のデータを検索して、見つからない場合は新規データをインサートし、見つかった場合はデータのアップデートをしています。  
書込みにはトランザクションブロックを使った方法もあるようですが、今回は`beginWriteTransaction`と`commitWriteTransaction`を利用しました。  

#####データの確認
では、データを作成した後に、そのデータを確認したい場合はどうすれば良いのでしょうか？  
実はMac専用のアプリが提供されています。  

[こちらのサイト](https://realm.io/jp/docs/java/latest/)の **Realm Browser** の項目からインストールしましょう。  

Realm Browserを使えば、下記のようにデータを見ることができます。  

![Realm Browserで確認１](/images/swift-realm4.png)  
![Realm Browserで確認２](/images/swift-realm5.png)  

これからSQLiteよりもRealmの方がスタンダートな保存方法となることでしょう。  
どんどん使い方を勉強していかなくては！！  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
