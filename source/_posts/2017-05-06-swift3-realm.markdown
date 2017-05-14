---
layout: post
title: "Swift3でRealmSwiftを使ってみよう！"
date: 2017-05-06 02:06
comments: true
categories: ios swift realm
---

### はじめに
約2年前に画期的なモバイルデータベースとして[Realm](https://realm.io/jp/)について紹介させて頂きました。  
当時はSwift専用のものがなかったため、Objective-C用のものをブリッジヘッダーファイルを作成することで利用していました。  
現在はかなり多くのアプリでも利用され、広く浸透していると共に、`SwiftRealm`が作られ、Swift専用化しています。  

今回は、以前、筆者が書いた[SwiftでRealmを使ってみよう！](http://grandbig.github.io/blog/2015/06/07/swift-realm/)を`SwiftRealm`で書き直す形でSwift3での`Realm`の使い方を見ていきたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### RealmSwiftの導入方法
`CocoaPods`を利用して導入してみます。  
(`Carthage`での利用方法も公式サイトにて紹介されています。)  

```objective-c
# Podfile
use_frameworks!

target "RealmSwiftSample" do
  # Normal libraries
  pod 'RealmSwift'

  abstract_target 'Tests' do
    inherit! :search_paths
    target "RealmSwiftSampleTests"
    target "RealmSwiftSampleUITests"
  end
end

post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['SWIFT_VERSION'] = '3.0'
    end
  end
end
```

因みに、`abstract_target`は複数targetにまたがって利用したいライブラリがある場合に利用します。  
(上記の例はテストでのみ利用するライブラリがある場合に利用する書式です。)  
また、`post_install do |installer|`〜`end`までの書式はSwiftのバージョンを指定するために追加します。  

`Podfile`ができたら`pod install`を実行して`xcworkspace`ファイルを開きましょう。  

### 保存オブジェクトの生成
Objective-C用のときは`RLMObject`型のクラスを作成していましたが、`RealmSwift`では単に`Object`型のクラスを作成します。  

```objective-c
// Engineer.swift

import Foundation
import RealmSwift

// Skillクラス
class Skill: Object {
    dynamic var name: String = ""
}

// Engineerクラス
class Engineer: Object {
    dynamic var id: Int = 0
    dynamic var name: String = ""
    dynamic var level: Int = 0
    let skills = List<Skill>()
    dynamic var created: Double = Date().timeIntervalSince1970
    dynamic var updated: Double = Date().timeIntervalSince1970

    // プライマリーキーの設定
    override static func primaryKey() -> String? {
        return "id"
    }

    // インデックスの設定
    override static func indexedProperties() -> [String] {
        return ["level"]
    }
}
```

Objective-C用の`Realm`では`RLMArray`型として`skills`を作成していましたが、`RealmSwift`では`List<Skill>`型として作成できます。  
こちらの方が直感的でわかりやすいですね。  
因みに、`dynamic var skills = List<Skill>()`と書くとエラーが発生するので`List`を利用する場合は`let`にしましょう。  

### オブジェクトのインサート/アップデート
先程作成したオブジェクトを保存する方法について紹介しましょう。  

```objective-c
// ViewController.swift
import UIKit
import RealmSwift

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        // 新規オブジェクトをインサート
        createEngineer(name: "test1", level: 1, skills: ["swift", "objective-c"])
    }

    <省略>

    func createEngineer(name: String, level: Int, skills: [String]) {
        // Skill型オブジェクトに変換してList<Skill>に格納
        let skillList = List<Skill>()
        for skill in skills {
            let newSkill = Skill()
            newSkill.name = skill
            skillList.append(newSkill)
        }

        let realm = try! Realm()
        
        // Engineer型オブジェクトの作成
        let engineer = Engineer()
        engineer.id = realm.objects(Engineer.self).count
        engineer.name = name
        engineer.level = level
        engineer.skills.append(objectsIn: skillList)

        // Realmへのオブジェクトの書き込み
        try! realm.write {
            realm.add(engineer)
        }
    }
}
```

Objective-Cの際は`realm.beginWriteTransaction()`や`realm.commitWriteTransaction()`などわざわざ書いていたものの、`RealmSwift`では非常にコンパクトに書けますね。  
因みに、アップデートであれば、  

```objective-c
try! realm write {
  engineer.name = "takahiro"
}
```

のようにすれば良さそうです。  

### データの確認
今は、Mac App Storeから`Realm Browser`アプリをインストールすることで簡単にデータ確認が可能になっています。  

手順は以下の通りです。  
１．`Realm Browser`を起動する  
２．`Realm`ファイルを選択して開く  

そうすることで、下記のようにデータを確認することができます。  

![Realm BrowserでEngineerオブジェクトを確認](/images/swift3-realm1.png)  
![Realm BrowserでSkillオブジェクトを確認](/images/swift3-realm2.png)  

注意すべきこととしては、シミュレータだと`realm`ファイルを探すのに骨が折れるかもくらいでしょうか...  
`/Users/<username>/Library/Developer/CoreSimulator/Devices/<simulator-uuid>/data/Containers/Data/Application/<application-uuid>/Documents/default.realm`にありますので該当するファイルを検索するなどして探しましょう。  

### まとめ
さて、如何でしたでしょうか。  
筆者は久しぶりに`Realm`を触ったため、以前利用していたときよりも『だいぶ変わったな』と正直思いました。  
ですが、`RealmSwift`になったことで、より`Swift`らしい書き方ができると思いますし、単純に記述量も少なく書けるような気がしています。  
今後も多くのアプリで利用されることでしょうし、知っておいて損は絶対になさそうですね。  
ということで本日はここまで。  

参考:  

* [CocoaPods でビルド設定を追加する](http://qiita.com/shu223/items/e9d0145a2087da0d6d46)  
* [CocoaPods公式サイトでの説明](https://guides.cocoapods.org/using/the-podfile.html)  
* [stackoverflow - How to find my realm file?](http://stackoverflow.com/questions/28465706/how-to-find-my-realm-file/28465803#28465803)  
* [Realm公式ドキュメント](https://realm.io/jp/docs/swift/latest/)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
