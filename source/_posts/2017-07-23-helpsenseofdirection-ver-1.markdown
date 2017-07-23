---
layout: post
title: "方向音痴メモ ver1.0の紹介"
date: 2017-07-23 11:27
comments: true
categories: ios MyApp
---

### はじめに
本日はついこの間まで趣味で開発していたアプリについて紹介したいと思います。  
それは[方向音痴メモ](https://itunes.apple.com/us/app/%E6%96%B9%E5%90%91%E9%9F%B3%E7%97%B4%E3%83%A1%E3%83%A2/id1260288529?mt=8&ign-mpt=uo%3D2)というアプリです。  
(仕事終わりに1時間ちびちびと開発していたこともあって、)製作期間は1ヶ月もかかってしまいました。  
全然大した機能があるわけでもないのに...。  

今回の開発を通して学んだこともあるため、記録として本記事を書こうと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### アプリの紹介
アプリの機能から紹介したいと思います。  
まず、アイコンはこちらになります。  

![方向音痴メモのアイコン](/images/HelpSenseOfDirection_icon.jpg)

#### アプリのコンセプト
このアプリのターゲットはアプリの名にもある通り『方向音痴な人』です。  
このターゲットを助けるために筆者は下記が必要だと考えました。  

- 道に迷った時に、通った場所の記憶を蘇らせるためのメモを残せる  
- メモは「その場所の概要」や「その場所の画像」を記録できるようにする  

上記を実装したのが、今回のアプリになります。  

#### アプリの使い方
ではアプリの使い方を見ていきましょう。  
今回はアプリ起動時にチュートリアルを見れるようにしたため、そのチュートリアルの指示に従って進めば簡単だと思います。  

##### 基本的な使い方
チュートリアルを進むことで、下記のように、基本的な使い方を知ることができます。  

![チュートリアル１](/images/HelpSenseOfDirection_tutorial_1.jpg)  
![チュートリアル２](/images/HelpSenseOfDirection_tutorial_2.jpg)  
![チュートリアル３](/images/HelpSenseOfDirection_tutorial_3.jpg)  

##### 不要なポイントの削除
誤ってポイントを作成してしまうこともあると思います。  
そんなときには下記のように不要なポイントを削除してしまいましょう。  

![不要なポイントの削除](/images/HelpSenseOfDirection_tutorial_4.jpg)  

##### ポイントの全削除
配置したポイントを一気に削除することもできます。  

![ポイントの全削除](/images/HelpSenseOfDirection_tutorial_5.jpg)  

##### チュートリアルをもう一度見る
初回起動時にチュートリアルを見たものの、使い方を忘れてしまうこともあるでしょう。  
そんな時用にチュートリアルをもう一度見る機能を用意しています。  

![チュートリアルをもう一度見る](/images/HelpSenseOfDirection_tutorial_6.jpg)  

### アプリの実装
続いて実装面に関して触れます。  
実コードは[GitHub: HelpSenseOfDirection](https://github.com/grandbig/HelpSenseOfDirection)を参照頂ければと思います。  

#### プロジェクト構成  
Xcode上のプロジェクト構成は下記の通りです。  
今回もMVCで実装しています。  

```objective-c
HelpSenseOfDirection
├── Enum
│    └── MarkerType.swift
├── Model
│    ├── CustomGMSMarker.swift
│    ├── Direction.swift
│    ├── Geocoding.swift
│    ├── RealmMarker.swift
│    ├── RealmMarkerManager.swift
│    └── Marker.swift
├── View
│    ├── MarkerInfoContentsView.xib
│    ├── MarkerInfoContentsView.swift
│    ├── UIPlaceHolderTextView
│    └── CustomCell.swift
├── ViewController
│    ├── AnnotationViewController.swift
│    ├── CreateMarkerViewController.swift
│    ├── ViewController.swift
│    ├── ViewController+CLLocationManager.swift
│    ├── ViewController+GMSMapView.swift
│    ├── ViewController+SpotlightControllerView.swift
│    ├── SettingViewController.swift
│    └── SlideMenuViewController.swift
├── AppDelegate.swift
└── Main.storyboard
```

#### 利用ライブラリ
今回利用したライブラリは下記になります。  

* [Realm / RealmSwift](https://github.com/realm/realm-cocoa/tree/master/RealmSwift)  
足跡のデータを端末内部に保存するために利用しています。  
* [Google Maps SDK for iOS](https://developers.google.com/maps/documentation/ios-sdk/?hl=ja)  
今回はAppleデフォルトではなく、Google Mapsを利用しています。  
* [Alamofire](https://github.com/Alamofire/Alamofire)  
Google Directions APIやGeocoding APIを利用する際にネットワーク通信が必要なので利用しています。  
* [SwiftyJSON](https://github.com/SwiftyJSON/SwiftyJSON)  
APIレスポンスとしてJSON方式で取得するので扱いやすさのために導入しています。  
* [Gecco](https://github.com/yukiasai/Gecco)  
チュートリアル表示用に利用しています。  

#### 各クラスの説明
今回も少々クラス説明を書いておきたいと思います。  

##### Enum/MarkerType.swift
今回のアプリでは **スタート地点/ゴール地点/途中ポイント地点** の3種類の地点にマーカを設置する必要があるため、`Enum`としてパターンを定義しています。  
デフォルトは最も多く設置するであろう **途中ポイント地点** である `MarkerType.point` としています。  
因みに `SwiftLint` の設定で `Enum` の `case` 定義は **小文字始まり** としているため、それぞれ `start/point/goal` としています。  

##### Model/CustomGMSMarker.swift
Google Maps SDK for iOSでは `GMSMarker` クラスが用意されています。  
今回のアプリでは、  

- スタート地点/ゴール地点/途中ポイント地点とタイプ別でマーカを設置する  
- `Realm`に`ID`を採番してマーカ情報を保存する  
- マーカをタップしたときに`InfoWindow`に表示する地点情報を`ID`を元に`Realm`から取得する  

という処理が必要であるため、通常の `GMSMarker` クラスでは機能が足りません。  
よってプロパティに`id`と`type`を追加した `CustomGMSMarker` を作成しました。   

##### Model/Direction.swift, Model/Geocoding.swift
こちらはGoogle Maps SDK for iOSだけではカバーできない機能があるため、  
`Google Directions API`や`Google Geocoding API`を利用します。  
そのためのクラスになります。  

##### Model/Marker.swift
これは少々わかりにくいクラスになってしまいました。  
と言うのも、`ViewController.swift`の`putMarker`のメソッドの引数を減らすためだけに作成したクラスだからです。  
各種で`Marker`というワードを利用していることもあって非常にわかりにくいですね...  

##### Model/RealmMarker.swift
`RealmSwift`を利用しているので、保存するモデルの形式です。  

##### Model/RealmMarkerManager.swift
`RealmSwift`を用いてデータの保存/取得/削除などを管理するクラスです。  

##### View/MarkerInfoContentsView.swift
マップにプロットするマーカをタップしたときに表示する`InfoWindow`に当たります。  

##### View/UIPlaceHolderTextView.swift
こちらは`InfoWindow`内に`UITextView`が必要だったのですが、`UILabel`同様に`placeholder`を表示したかったため利用しています。  
[UITextViewでのPlaceHolder（プレースホルダ）をSwiftで実装する方法](http://qiita.com/matsuhisa_h/items/5f4877e8ec89729de824)からほぼほぼ拝借させて頂きました。  

##### View/CustomCell.swift
チュートリアルを再度閲覧できるように設定画面に`UISwitch`つきの`Cell`を用意する必要があったため、作成しました。  

##### ViewController/ViewController+◯◯.swift
さて今回は`ViewController+CLLocationManager.swift`のように幾つかファイルを分けています。  
理由としては、`ViewController`が肥大化することで最大行数が`SwiftLint`のデフォルト値を超過してしまったためです。  
簡単のために`delegate`系を別ファイルとして切り出しました。  

### まとめ
さて、如何でしたでしょうか？  
正直、iOSアプリを開発する際のプロジェクト構成やアーキテクチャには迷いがあります。  
今は個人で1ヶ月程度で開発成果を残していくことが目的になっているため、慣れているMVC形式での開発で進めてしまっています。  
ただ、チーム開発やモダンな開発のことを考えるともう少しチャレンジを入れてみたいと思っています。  

引き続き別アプリの開発を進めているので、徐々に新しい要素を追加してブログ記事に残せていければと思います。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
