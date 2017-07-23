---
layout: post
title: "足跡計 ver1.0.3の紹介"
date: 2017-06-29 22:50
comments: true
categories: ios MyApp
---

### はじめに
本日はいつもと趣向を変えて、最近作り直したアプリについて紹介したいと思います。  
それは[足跡計](https://itunes.apple.com/jp/app/%E8%B6%B3%E8%B7%A1%E8%A8%88/id725412788?mt=8)というアプリです。  
これは実に3年半以上昔に作成したアプリで、v1.0.0リリース後に一度も更新をしていませんでした...。  

そんな状況なので当たり前ではあるのですが、  
前々からAppleより警告が来ていたものの対応せずにいたら、とうとうApp Storeから削除されてしまいました。　　

最近、iOSアプリ開発を仕事でもする機会がなくなっていた筆者にとって、これを良い機会だと捉え、アプリを作り直してみることにしました。  

<a href="https://itunes.apple.com/jp/app/zu-ji-ji/id725412788?mt=8&uo=4" target="itunes_store"style="display:inline-block;overflow:hidden;background:url(https://linkmaker.itunes.apple.com/htmlResources/assets/ja_jp//images/web/linkmaker/badge_appstore-lrg.png) no-repeat;width:135px;height:40px;@media only screen{background-image:url(https://linkmaker.itunes.apple.com/htmlResources/assets/ja_jp//images/web/linkmaker/badge_appstore-lrg.svg);}"></a>

<!-- more -->

### アプリの紹介
まずはアプリの機能について紹介してみたいと思います。  
基本的にはv1.0.0からそんなに変えてはいません。  

アイコンはこちらになります。  

![足跡計のアイコン](/images/footStepMeter_Icon.png)  

#### 移動経路の計測
メイン機能として **移動経路の計測** が可能です。  
手順は実に簡単で、  

1. 初期画面にてタブの『開始』ボタンをタップします  
2. 計測したい精度を選択します  
3. 計測する経路のタイトルを設定します  

以上となります。  

![移動経路の計測手順](/images/footStepMeter4.png)  

計測したい精度の種類としては、  

* 最高精度: kCLLocationAccuracyBestForNavigation  
* 高精度: kCLLocationAccuracyBest  
* 10m誤差: kCLLocationAccuracyNearestTenMeters  
* 100m誤差: kCLLocationAccuracyHundredMeters  
* 1km誤差: kCLLocationAccuracyKilometer  
* 3km誤差: kCLLocationAccuracyThreeKilometers  

となっています。  

#### 過去経路の閲覧
計測直後の経路を見ることはもちろん、過去の経路を閲覧することもできます。  
また、経路表示のために配置しているアイコンを「人の足跡アイコン」と「動物の足跡アイコン」の2つを切り替えることが可能です。  

さらに、もう1つの機能として、過去経路のデータをCSVファイルとしてメールで送信することができます。  

![過去の経路の閲覧](/images/footStepMeter5.png)  

### アプリの実装
さて、続いて、アプリの実装部分、中身について紹介したいと思います。  
実コードは[GitHub: footStepMeter](https://github.com/grandbig/footStepMeter)を見て頂ければわかるのですが、かいつまんで少々説明したいと思います。  

#### プロジェクト構成  
プロジェクト構成は下記の通りです。  
シンプルにMVCで実装しています。  

```objective-c
footStepMeter
├── Enum
│    └── LocationAccuracy.swift
├── Model
│    ├── Location.swift
│    ├── Footprint.swift
│    ├── FootprintManager.swift
│    ├── CustomAnnotation.swift
│    └── UIImage+Extension.swift
├── View
│    ├── PickerView.xib
│    └── PickerView.swift
├── AppDelegate.swift
├── ViewController.swift
├── SettingViewController.swift
├── FootprintsViewController.swift
├── AboutViewController.swift
├── LicenceViewController.swift
├── HistoryViewController.swift
└── Main.storyboard
```

本当は `ViewController`系をフォルダにまとめても良かったのですが、そのままにしています。  
あとMain.storyboardも`View`フォルダ配下においても良かったのですが、これもそのままにしています。  

#### 利用OSSライブラリ
今回導入したOSSライブラリは以下の通りです。  

* [Realm / RealmSwift](https://github.com/realm/realm-cocoa/tree/master/RealmSwift)  
足跡のデータを端末内部に保存するために利用しています。  
因みに、V1.0.0では[fmdb](https://github.com/ccgus/fmdb)を使っていました。  
* [Quick](https://github.com/Quick/Quick)  
UIテストのために利用しています。

#### 各クラスの説明
少し、各クラスの説明や意図についても紹介します。  

##### Enum/LocationAccuracy.swift
わざわざ `Enum/LocationAccuracy.swift` を用意した理由としては、位置情報の精度は各種クラスで利用する可能性があるためです。  
実際は `Model/Location.swift` と `View/PickerView.swift` で利用しています。  

##### Model/Location.swift
位置情報関連のロジック処理を書いています。  
実態は `CLLocationManagerDelegate` を逃した感じになっています。  

##### Model/Footprint.swift
`RealmSwift`を利用しているので、保存するモデルの形式ですね。  

##### Model/FootprintManager.swift
`RealmSwift`を用いてデータの保存/取得/削除などを管理するクラスです。  

##### View/PickerView.swift
`UIPickerView`は`UITableView`に負けず劣らず面倒な作業が多いので、`ViewController`からは切り離して扱っています。  
`Main.storyboard`に配置することはできなくもないのですが、  

* 必要な機会が少ないのに、`UITabBar`などの他の要素と被って配置されるのが気になる  
* 初めは非表示状態にしたい  

の理由から `ViewController` で `self.view.addSubView(pickerView!)` として要素を追加しています。  

位置情報の精度を選択するときに利用するので、ピッカーの各行に精度を表示する必要があります。  
そこで `Enum/LocationAccuracy.swift` で定義した値を利用しています。   

### アプリのテストについて
`Quick`を導入した本格的なテストを実装することを考えていたものの、実際にはあまり書けていません...  

`FootprintManager.swift`の単体テスト用に`FootprintManagerTests.swift`を実装しました。  
中身は[QuickでSwiftコードのUnitテストをしよう！(2)](http://grandbig.github.io/blog/2017/05/06/quick-2/)で書いた内容です。  

View系のテストも実装しようと思ったものの、下記のように途中までしか書けていません。  
(というのも、View系だとUIテストの方に譲った方が実装しやすいのかなと思ったからなんですよね。)  

```objective-c
// ViewControllerTests.swift
import Quick
import Nimble
import RealmSwift
@testable import footStepMeter

class ViewControllerTests: QuickSpec {
    override func spec() {
        var subject: ViewController!

        beforeEach {
            // StoryboardからViewControllerを初期化
            subject = UIStoryboard(name: "Main", bundle: nil).instantiateViewController(withIdentifier: "ViewController") as? ViewController

            expect(subject.view).notTo(beNil())
            expect(subject.statusBarView).notTo(beNil())
            expect(subject.navigationBar).notTo(beNil())
            expect(subject.mapView).notTo(beNil())
            expect(subject.tabBar).notTo(beNil())
            expect(subject.currentLocationButton).notTo(beNil())
            expect(subject.countLabel).notTo(beNil())
        }

        it("countLabel default is ****") {
            expect(subject.countLabel.text).to(equal("****"))
        }

        it("User's current location move when tapped currentLocationButton") {
            subject.currentLocationButton.sendActions(for: UIControlEvents.touchUpInside)
            print(subject.mapView.region.span.latitudeDelta)
            print(subject.mapView.region.span.longitudeDelta)
            expect(String(format: "%.2f", subject.mapView.region.span.latitudeDelta)).to(equal("0.06"))
            expect(String(format: "%.2f", subject.mapView.region.span.longitudeDelta)).to(equal("0.05"))
        }
    }
}
```

UIテストもRecording機能を使って、少し実装したものの、やっぱり手で書かなくてはいけない部分が出てくるな〜と思い、一旦止めています。  
(ま、当たり前なんですけどね。)  

### まとめ
さて如何でしたでしょうか？  
今回はiOSアプリ開発およびSwiftコーディングのリハビリも兼ねて進めてみました。  
また別のアプリ開発も考えていたりするので、完成でき次第、どんどんブログでも紹介できればと思います。  

と言ったところで本日はここまで。  

<a href="https://itunes.apple.com/jp/app/zu-ji-ji/id725412788?mt=8&uo=4" target="itunes_store"style="display:inline-block;overflow:hidden;background:url(https://linkmaker.itunes.apple.com/htmlResources/assets/ja_jp//images/web/linkmaker/badge_appstore-lrg.png) no-repeat;width:135px;height:40px;@media only screen{background-image:url(https://linkmaker.itunes.apple.com/htmlResources/assets/ja_jp//images/web/linkmaker/badge_appstore-lrg.svg);}"></a>
