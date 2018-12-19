---
layout: post
title: "5年前に初めて個人開発したアプリを再構築している話"
date: 2018-12-12 00:00
comments: true
categories: ios swift MyApp rx
---

### はじめに
今年もやってきましたAdvent Calendarの季節！  
こちらは[個人開発 #2 Advent Calendar 2018](https://qiita.com/advent-calendar/2018/private-developmen2)の12日目の記事です。  

今年は、5年前に初めて個人開発したアプリを再構築している話を思い出を交えながら書こうと思います。  
とは言え、絶賛再構築中なので、恐らく本記事公開日までに作り終わらないと思うのですが、それも一興ということで大目に見て頂ければと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### 5年前になぜ個人開発をしようと思ったのか
まずは筆者がなぜ個人開発を始めようと思ったのかについて話したいと思います。  
当時の筆者の状況やスキルを思い出してみます。  

#### 状況

* 社会人3年目  
* 当然、役職も重い責任も何もないぺーぺー  
* プロジェクトを1〜2個経験(内の1つは大炎上を経験)  

#### スキル

* `HTML5, CSS3, jQuery`を利用したWEBフロントエンドの開発  
* `iOS, Android`アプリの開発  
* `Node.js, MongoDB`を用いたサーバサイドの開発  

#### 状況とスキルを見た振り返り
一見、スキルを見ると凄そうに見えるかもしれませんが、  
`iOS, Android`アプリの中身は`WebView`を利用しており、ネイティブコードは位置情報の取得やプッシュ通知などのごく一部でした。  

また、当時、非常に新しかった`Node.js`や`MongoDB`も訳が分からず利用しており、  
その知識やスキル不足からプロジェクトを大炎上させてしまいました。  

その時に学んだ重要なことは、  

* メモリ消費を考えて、何でもかんでも `DB` からデータを引っ張ってくるのは辞めよう  
* CPU消費を考えて、大量データを永遠と`for`ループ回すのは辞めよう  

といったエンジニアとしては『当たり前 & これができなかったらごめんなさいで済まされないレベル』の話でした。  

#### 個人開発を始めた理由
そんな状態の私がなぜ個人開発を始めたのかというと...  

1. 技術的な強みを1つ持ちたかった  
2. エンジニアとしての将来が不安で悶々としていた  

という2つが大きな理由です。  

理由2を満たすために、手始めに技術ブログを開設したのですが、  
『どうせなら理由1も満たしたい！』という想いから、  
当時最も興味のあった `iOS` アプリの開発を勉強しながら始めることにしました。  
※ `iOS` ネイティブアプリの開発ができるようになるというのが当面の目標でした。  
( `WebView`ではなく、ネイティブアプリの開発です。 )

### 足跡計について
初めのうちは基本的なXcodeの使い方などを試してブログに書いたりしていたのですが、  
折角なので、1つアプリを完成させて世にリリースしたいという想いが強くなりました。  
そうして完成した個人開発の第一段階アプリが『[足跡計](https://itunes.apple.com/jp/app/%E8%B6%B3%E8%B7%A1%E8%A8%88/id725412788?mt=8)』です。  

#### 足跡計の機能
このアプリには、次のような機能を持たせました。  

* 様々な精度で歩行ルートを記録可能  
* 複数の歩行ルートを記録可能  
* 歩行ルート履歴をいつでも閲覧可能  
* 歩行ルート記録をメールで送信可能  
* 不要になった歩行ルート記録は削除可能  

このアプリを開発しようと思った理由は、  
当時、業務にて位置情報を扱ったアプリ内 `WebView` のアプリを開発・運用しており、  
位置情報機能は私にとって非常に扱いやすかったためです。  

【足跡計のスクリーンショット】  
![足跡計のスクリーンショット](/images/personal_development_footprintmeter_1.png)

#### 今回再構築をしようと思った理由
さて、ここまでは5年前の個人でのアプリ開発に至るまでの話をしてきました。  
ここからが本題で、今回、筆者は思い切って、足跡計アプリを再構築しようと考えました。  

その理由は、  

1. iPhoneXの画面にアプリが対応できておらず、格好悪い(上下の黒帯の幅が長い)  
2. ホームアイコンとアプリ画面の色合いを統一させたい  
3. `MVVM` アーキテクチャを用いたアプリ開発を試したい  

という3つの想いがあったためです。  

特に『3』に関しては、  
筆者が業務で昨年から今年の春先にかけて `Clean Swift` アーキテクチャでのiOSアプリ開発に挑戦し、改めてiOSアプリのアーキテクチャのあり方に悩んだことが元となっています。  

具体的に悩んだ内容に関しては後日別途書こうと思いますが、  
上記経緯より、現在最も頻繁に採用されているであろう [RxSwift](https://github.com/ReactiveX/RxSwift) を用いた `MVVM` アーキテクチャ設計をきちんと勉強しておきたいと思わずにはいられなかったのです。  

では、前段はここまでとして、ここから先は、  

* どのように設計を変更したのか(プロジェクト構成の話)  
* `RxSwift` を用いた `MVVM` アーキテクチャで構成されたメインロジックの説明  
* その上で躓いたこと / ハマったこと  

を中心に説明し、最後に今後の展望とリリース時期を説明して終了にしたいと思います。  

### 足跡計の再構築について
では、再構築について一つずつ説明してきましょう。  

#### プロジェクト構成の変更
まずはアーキテクチャの変更によって生じたプロジェクト構成の変更について説明します。  
これまでは `MVC` アーキテクチャを採用していたため、下記のようなプロジェクト構成になっていました。  

```objective-c
// 再構築前のMVCアーキテクチャでのプロジェクト構成
footStepMeter
├── Enum
│    └── LocationAccuracy.swift
├── Model
│    ├── CustomAnnotation.swift
│    ├── Footprint.swift
│    ├── FootprintManager.swift
│    ├── Location.swift
│    └── UIImage+Extension.swift
├── View
│    └── PickerView.swift
├── Main.storyboard
├── AppDelegate.swift
├── ViewController.swift
├── SettingViewController.swift
...
```

一応、フォルダ分けして責務を見やすくしようとした形跡がありますが、  
下記観点が全然ダメだなと今振り返って思います。  

* `Model` の責務がカオスになりかけている  
  * `UIImage+Extension` は `Model` 配下でなくても良いはず  
* `ViewController` がフォルダ分けされておらず、ただ並んでいる  
  * 今後、画面が増えてきた時に `ViewController` も増えるので視認性が悪くなる   

今回は、 `MVVM` アーキテクチャを採用し、プロジェクト構成も見直しています。  

```objective-c
footStepMeter
├── Enum
│    ├── AlertActionType.swift
│    ├── LocationAccuracy.swift
│    └── TabBarItemTag.swift
├── Model
│    ├── CustomAnnotation.swift
│    ├── Footprint.swift
│    └── RealmManager.swift
├── View
│    ├── Parts
│    │    ├── CustomTableViewCell
│    │    └── PickerView
│    ├── Map
│    │    ├── MapViewController.swift
│    │    └── MapViewController.xib
│    ├── Setting
│    ...
├── ViewModel
│    ├── MapViewModel.swift
│    ├── SettingViewModel.swift
│    ...
├── Protocol
│    └── Injectable.swift
├── Extensions
│    ├── UIImage+Extension.swift
│    ├── UIViewController+Rx.swift
│    ...
├── AppDelegate.swift
...
```

変更点としては、  

* `MVVM` なので、 `View` / `Model` / `ViewModel` フォルダを作る  
* `Extension` 系は `Extensions` フォルダを作ってまとめる  
* 疎結合の肝となる `Protocol` も専用に `Protocol` フォルダを作る  
* `Storyboard` を廃止し、 `xib` を採用  

となります。  
設計思想的な面で変更している部分はあるものの、以前に比べれば視認性は上がったと思っています。  
(もう少し改善の余地はあるなと思いつつ...)  

#### RxSwiftを用いたMVVMアーキテクチャでの構成
続いて、 `RxSwift` を用いた `MVVM` アーキテクチャで具体的に何をどのように書いているのか紹介したいと思います。  
全ては紹介しきれないため、メイン画面であるマップ画面を元に一部を説明したいと思います。  

##### マップ画面の機能
具体的にスクショを交えながら、機能を紹介します。  

![歩行ルートの保存開始機能](/images/personal_development_footprintmeter_2.png)  

![歩行ルートの保存終了機能](/images/personal_development_footprintmeter_3.png)  

![歩行ルートの表示/非表示切替機能](/images/personal_development_footprintmeter_4.png)  

さて、ここからはソースコードベースで実装について説明したいと思います。  

##### View / Model / ViewModelそれぞれの責務
先程スクショベースでお見せした機能をロジックベースで言語化すると、  

* 位置情報の取得許可の確認  
* 位置情報の取得情報の確認  
* 位置情報の計測を開始し、`Realm` にそのデータを保存する  
* 位置情報の計測を停止する  
* `Realm` から保存した位置情報を取得する  

のように言い換えられます。  

では、 `View` / `Model` / `ViewModel` それぞれどんな責務を持たせれば良いのでしょうか。  
下記にそれぞれの責務を簡単に書き出してみました。  

* `Model`  
  * いわゆるビジネスロジックを担当する  
  * 例) API関連、ローカルDBを扱い関連など
* `View`  
  * ユーザアクションのキャッチ  
  * 画面の描画   
* `ViewModel`  
  * `View` と `Model` を繋ぐ  
  * `View` からの処理依頼を受けて、`Model`を介して必要な情報を取得し、`View`に特定の描画司令を出す  

続いて、具体的に上記を実現する方法について説明します。  

##### Modelの説明
まずは `Model` に関する実装から説明します。  
本アプリの肝となる『計測した位置情報の `Realm` への保存』を実装するために、  
`Realm` を管理する `RealmManager` を定義します。  

また、 `Realm` に保存する形式を先に決める必要があるため、 `Footprint` という `Model` を作成します。  

以下、 `Footprint` のソースコードです。  
書式は `Realm` の使い方そのままなので詳細は省きます。  

```objective-c
// Model/Footprint.swift
import RealmSwift

/// 足跡
class Footprint: Object {
    // ID
    @objc dynamic var id: Int = 0
    // 保存した歩行ルートのタイトル
    @objc dynamic var title: String = ""
    // 緯度
    @objc dynamic var latitude: Double = 0.0
    // 軽度
    @objc dynamic var longitude: Double = 0.0
    // 位置の精度
    @objc dynamic var accuracy: Double = 0.0
    // 歩行速度
    @objc dynamic var speed: Double = 0.0
    // 歩行方向
    @objc dynamic var direction: Double = 0.0
    // データの生成日時
    @objc dynamic var created: Double = Date().timeIntervalSince1970

    // プライマリーキーの設定
    override static func primaryKey() -> String? {
        return "id"
    }

    // インデックスの設定
    override static func indexedProperties() -> [String] {
        return ["title"]
    }
}
```

続いて、 `RealmManager` のソースコードです。  
まずは、 `protocol` として `RealmManagerClient` を定義します。  
実際の `RealmManager` クラスは `RealmManagerClient protocol` を継承します。  

こうすることで、テストを書く際にモックデータを返却することが容易になります。   

```objective-c
// Model/RealmManager.swift
import Foundation
import CoreLocation
import RxSwift
import RealmSwift

protocol RealmManagerClient {
    // MARK: - Protocol Properties
    var title: String { get set }

    // MARK: - Protocol Methods
    func setSaveTitle(_ title: String)
    func createFootprint(location: CLLocation)
    func fetchFootprints() -> Observable<Results<Footprint>?>
    func fetchFootprintsByTitle(_ text: String) -> Observable<Results<Footprint>?>
    func existsByTitle(_ text: String) -> Observable<Bool>
    func countFootprints() -> Observable<Int>
    func countFootprintsByTitle(_ text: String) -> Observable<Int>
}
```

この `RealmManagerClient protocol` を継承して各メソッドの実処理を実装すると、下記のようになります。  

```objective-c
// Model/RealmManager.swift
final class RealmManager: NSObject, RealmManagerClient {

    // MARK: - Properties
    var title = String()

    // MARK: - Initial Methods
    override init() {
        super.init()
    }

    /// タイトルの保存処理
    ///
    /// - Parameter title: 保存したいタイトル
    func setSaveTitle(_ title: String) {
        self.title = title
    }

    // MARK: - CRUD

    /// 位置情報のデータの保存処理
    ///
    /// - Parameter location: 保存する位置情報
    func createFootprint(location: CLLocation) {
        do {
            let realm = try Realm()
            let footprint = Footprint()
            let savedLastFootprint = fetchAllFootprints()?.last
            footprint.id = (savedLastFootprint != nil) ? ((savedLastFootprint?.id)! + 1) : 0
            footprint.title = self.title
            footprint.latitude = location.coordinate.latitude
            footprint.longitude = location.coordinate.longitude
            footprint.accuracy = location.horizontalAccuracy
            footprint.speed = location.speed
            footprint.direction = location.course

            // Realmへのオブジェクトの書き込み
            try realm.write {
                realm.create(Footprint.self, value: footprint, update: false)
            }
        } catch let error as NSError {
            print("Error: code - \(error.code), description - \(error.description)")
        }
    }

    /// 保存している全位置情報データを取得する処理
    ///
    /// - Returns: 保存している全位置情報データ
    func fetchFootprints() -> Observable<Results<Footprint>?> {
        let footprints = fetchAllFootprints()
        return Observable.just(footprints)
    }

    /// 指定したタイトルで保存されている位置情報データを取得する処理
    ///
    /// - Parameter text: タイトル
    /// - Returns: 指定したタイトルで保存されている位置情報データ
    func fetchFootprintsByTitle(_ text: String) -> Observable<Results<Footprint>?> {
        do {
            let realm = try Realm()
            let footprints = realm.objects(Footprint.self).filter("title == '\(text)'")
            if footprints.count > 0 {
                return Observable.just(footprints)
            }
            return Observable.just(nil)
        } catch _ as NSError {
            return Observable.just(nil)
        }
    }

    /// 指定したタイトルで保存されている位置情報データがあるか確認する処理
    ///
    /// - Parameter text: タイトル
    /// - Returns: 存在する場合はtrue, 存在しない場合はfalseを返却する
    func existsByTitle(_ text: String) -> Observable<Bool> {
        do {
            let realm = try Realm()
            let footprints = realm.objects(Footprint.self).filter("title == '\(text)'")
            if footprints.count > 0 {
                return Observable.just(true)
            }
            return Observable.just(false)
        } catch _ as NSError {
            return Observable.just(false)
        }
    }

    /// 保存したい全位置情報の数を取得する処理
    ///
    /// - Returns: 保存している位置情報の数
    func countFootprints() -> Observable<Int> {
        do {
            let realm = try Realm()
            return Observable.just(realm.objects(Footprint.self).count)
        } catch _ as NSError {
            return Observable.just(0)
        }
    }

    /// 指定したタイトルで保存されている位置情報の数
    ///
    /// - Parameter text: タイトル
    /// - Returns: 保存している位置情報の数
    func countFootprintsByTitle(_ text: String) -> Observable<Int> {
        do {
            let realm = try Realm()
            let footprints = realm.objects(Footprint.self).filter("title == '\(text)'")
            return Observable.just(footprints.count)
        } catch _ as NSError {
            return Observable.just(0)
        }
    }

    // MARK: - Private Methods

    /// 保存している全位置情報データを取得する処理
    ///
    /// - Returns: 位置情報データ
    private func fetchAllFootprints() -> Results<Footprint>? {
        do {
            let footprints = try Realm().objects(Footprint.self).sorted(byKeyPath: "id")
            return footprints
        } catch _ as NSError {
            return nil
        }
    }
}
```

##### protocol Injectableを用意することで依存関係の解決
続いて、`ViewModel` や `View` の説明をする前に、  
`ViewModel` と `View` の双方を疎結合にするための `protocol Injectable` を定義します。  
※これは[WEB+DB PRESS V.106](https://gihyo.jp/magazine/wdpress/archive/2018/vol106)で特集されていた手法をそのまま採用しています。  

```objective-c
import UIKit

protocol Injectable {
    associatedtype Dependency
    init(with dependency: Dependency)
}

extension Injectable where Dependency == Void {
    init() {
        self.init(with: ())
    }
}
```

実際の効力は `ViewModel` や `View` のソースを見て頂けると伝わるかと思います。   

##### ViewModelの説明
では、`ViewModel`について次は見ていきます。  

先程言語化した  

* 位置情報の取得許可の確認  
* 位置情報の取得情報の確認  
* 位置情報の計測を開始し、`Realm` にそのデータを保存する  
* 位置情報の計測を停止する  
* `Realm` から保存した位置情報を取得する  

の5つを実装の内の幾つかを例に説明していきます。  

繰り返しになりますが、 `ViewModel` ですので、以下を守ることを念頭に置くことが大事です。  

* `ViewModel` の責務  
  * `View` と `Model` を繋ぐ  
  * `View` からの処理依頼を受けて、`Model`を介して必要な情報を取得し、`View`に特定の描画司令を出す  

まずは、 `ViewModel` の最低限の実装から先に説明します。  

```objective-c
// MapViewModel.swift
import Foundation
import RxSwift
import RxCocoa
import CoreLocation
import RealmSwift

// 説明(1)
final class MapViewModel: Injectable {

    // 説明(2)
    struct Dependency {
        let locationManager: CLLocationManager
        let realmManager: RealmManagerClient
    }

    // MARK: - Properties
    private let disposeBag = DisposeBag()

    // MARK: Initial method
    // 説明(3)
    init(with dependency: Dependency) {
        let locationManager = dependency.locationManager
        let realmManager = dependency.realmManager
        ...
    }
}
```

**説明(1)**  
`MapViewModel` クラスは `Injectable` プロトコルを継承するクラスとして定義します。  

**説明(2)**  
`Injectable` は `Generic Protocol` として定義されているため、
説明(1)の実装により、 `Dependency` を定義する必要が出てきます。  
ここでは `struct` として、そのプロパティに  

* `CLLocationManager` 型の `locationManager`  
* `RealmManagerClient` 型の `realmManager`  

を定義しています。

ミソなのが、 `RealmManager` ではなく `RealmManagerClient` としている点です。  
`RealmManagerClient` は `protocol` なので、具体的な処理は書かれていません。  
あくまでもインタフェースの提供のみです。  

このため、テストを書く際に、レスポンスをモック化することが容易になるのです。  
※ `CLLocationManager` はApple提供の純正品なので難しいですが...  

**説明(3)**  
`MapViewModel` の初期化メソッドの引数に `Dependency` 型の `dependency` を渡しています。  
初期化時の引数として外部から渡せるようにすることで依存性を軽減しています。  

説明(2)の実装を活かすために、外部から渡せるようにしたと言いかえることもできますね。  

続いて、 `View` と `ViewModel` を繋ぐ `RxSwift` の実装部分を説明してきます。  

5つの実装の内の  

* 位置情報の取得許可の確認  
* 位置情報の取得情報の確認  

は下記の通りに実装しています。  

```objective-c
// MapViewModel.swift
import Foundation
import RxSwift
import RxCocoa
import CoreLocation
import RealmSwift

final class MapViewModel: Injectable {

    struct Dependency {
        let locationManager: CLLocationManager
        let realmManager: RealmManagerClient
    }

    // MARK: - Properties
    private let disposeBag = DisposeBag()

    // MARK: Drivers
    private (set) var authorized: Driver<Bool>
    private (set) var location: Driver<CLLocationCoordinate2D>

    // MARK: Initial method
    init(with dependency: Dependency) {
        let locationManager = dependency.locationManager
        let realmManager = dependency.realmManager

        // Initialize stored properties
        // 位置情報の取得許可の確認
        authorized = Observable.deferred({() -> Observable<CLAuthorizationStatus> in
            let status = CLLocationManager.authorizationStatus()
            return locationManager
                .rx.didChangeAuthorizationStatus
                .startWith(status)
        })
            .asDriver(onErrorJustReturn: CLAuthorizationStatus.notDetermined)
            .map {
                switch $0 {
                case .authorizedAlways:
                    return true
                default:
                    return false
                }
        }

        // 位置情報の取得情報の確認
        location = locationManager.rx.didUpdateLocations
            .asDriver(onErrorJustReturn: [])
            .flatMap {
                return $0.last.map(Driver.just) ?? Driver.empty()
            }
            .map {
                realmManager.createFootprint(location: $0)
                return $0.coordinate
        }

        // 位置情報の取得許可を要求
        locationManager.requestAlwaysAuthorization()
        // バックグラウンドでの位置情報取得を許可
        locationManager.allowsBackgroundLocationUpdates = true
        // バックグラウンドで位置情報取得がわかるように設定
        locationManager.showsBackgroundLocationIndicator = true
    }
}
```

これは[RxSwiftの公式ExampleのGeolocationService](https://github.com/ReactiveX/RxSwift/blob/master/RxExample/RxExample/Services/GeolocationService.swift)と同じ実装です。  

詳細は[GeolocationSampleから学ぶdelegateのRx対応](http://grandbig.github.io/blog/2018/10/06/rx-delegate/)でも説明しているので、ここでは概略だけにします。  

* 位置情報の補足等を `Rx` でできるように独自に実装する必要があります  
* これにより `didChangeAuthorizationStatus` と `didUpdateLocations` を `locationManager.rx.xxx` のように `Rx` 実装方式に則って書けるようになります  
* それぞれハンドリングした値を `authorized` と `location` に渡すことで `View` で検知できるようにします  

また、5つの実装の内の  

* 位置情報の計測を開始し、`Realm` にそのデータを保存する  
* 位置情報の計測を停止する  

は下記の通りに実装しています。  

```objective-c
// MapViewModel.swift
import Foundation
import RxSwift
import RxCocoa
import CoreLocation
import RealmSwift

final class MapViewModel: Injectable {
    ...
    // MARK: - Properties
    private let disposeBag = DisposeBag()
    private var dataTitle = String()
    private var isUpdatingLocation = false

    // 説明(4)
    // MARK: PublishSubjects
    private let startUpdatingLocationStream = PublishSubject<(LocationAccuracy, String?)>()
    private let stopUpdatingLocationStream = PublishSubject<Void>()

    // MARK: BehaviorRelays
    private let errorStream = BehaviorRelay<String?>(value: nil)

    // MARK: Initial method
    init(with dependency: Dependency) {
        ...
        // Data Binding Handling
        // 説明(6)
        observeStartUpdatingLocation(locationManager: locationManager, realmManager: realmManager)
        observeStopUpdatingLocation(locationManager: locationManager)
    }
}

// 説明(5)
// MARK: - Input
extension MapViewModel {

    var startUpdatingLocation: AnyObserver<(LocationAccuracy, String?)> {
        return startUpdatingLocationStream.asObserver()
    }
    var stopUpdatingLocation: AnyObserver<Void> {
        return stopUpdatingLocationStream.asObserver()
    }
}

// MARK: - Output
extension MapViewModel {

    var error: Driver<String?> {
        return errorStream.asDriver()
    }
}

// MARK: - Data Binding Handling
// 説明(6)
extension MapViewModel {

    /// startUpdatingLocationStreamにデータバインディングされてきた場合の処理
    ///
    /// - Parameters:
    ///   - locationManager: 位置情報管理マネージャ
    ///   - realmManager: Realm管理マネージャ
    func observeStartUpdatingLocation(locationManager: CLLocationManager, realmManager: RealmManagerClient) {

        startUpdatingLocationStream
            .subscribe { [weak self] event in
                guard let strongSelf = self, let element = event.element, let dataTitle = element.1 else { return }
                strongSelf.dataTitle = dataTitle
                let locationAccuracy = LocationAccuracy.toCLLocationAccuracy(element.0)
                // タイトルの設定
                realmManager.setSaveTitle(dataTitle)
                // 同名タイトルの既存データが存在するか確認
                realmManager.existsByTitle(dataTitle)
                    .flatMapLatest({ isExist -> Observable<String?> in
                        if isExist {
                            return Observable.just(R.string.mapView.alreadySameTitleErrorMessage())
                        }
                        // 位置情報の取得精度を設定
                        locationManager.desiredAccuracy = locationAccuracy
                        // 位置情報の計測を開始
                        locationManager.startUpdatingLocation()
                        strongSelf.isUpdatingLocation = true
                        return Observable.just(nil)
                    })
                    .asDriver(onErrorJustReturn: R.string.mapView.unExpectedErrorMessage())
                    .drive(strongSelf.errorStream)
                    .disposed(by: strongSelf.disposeBag)
            }
            .disposed(by: disposeBag)
    }

    /// stopUpdatingLocationStreamにデータバインディングされてきた場合の処理
    ///
    /// - Parameter locationManager: 位置情報管理マネージャ
    func observeStopUpdatingLocation(locationManager: CLLocationManager) {

        stopUpdatingLocationStream
            .subscribe { [weak self] _ in
                guard let strongSelf = self else { return }
                // 位置情報の計測を停止
                locationManager.stopUpdatingLocation()
                strongSelf.isUpdatingLocation = false
            }
            .disposed(by: disposeBag)
    }
}
```

**説明(4)**  
`View` からの位置情報の取得開始と停止イベント通知をキャッチした後に、 `ViewModel` 内の処理に導くために実装している部分になります。  
開発当初に `Observable` であり `Observer` でもある `PublishSubject` を利用する必要があったため、 `PublishSubject` 型として宣言しています。  
現段階では `Observer` で十分な気がします。  
(今後の宿題と言うことで...)  

**説明(5)**  
`startUpdatingLocationStream` と `stopUpdatingLocationStream` を `PublishSubject` として定義したことで、必要となった実装です。  
`Observable` であり `Observer` でもある `PublishSubject` は便利な反面、`public` なプロパティとしておくと、誤って外側から `Observable` な機能を利用される可能性があります。  

これを防ぐために `AnyObserver` 型のプロパティを外部に公開し、`PublishSubject` 型プロパティは `private` として内部に閉ざしています。  

因みに、 `Input` / `Output` と分けて書くことで視認性が高まるので、  
`error` に至っては `BehaviorRelay` 関連にも関わらず、この書式を取っています。  
※ `error` は `Output` 時のみの利用かつ、`ObservableType` 型の `BehaviorRelay` なので誤った利用がされる恐れはないため。  

**説明(6)**  
実際に `View` からの指示を受け取った後に実行している処理になります。  
この中で、必要な情報を `Model` を介して取得することで `MVVM` というアーキテクチャが取れているわけです。  
`ViewModel` の中で、 `View` の描画に必要な情報を整理して、必要な情報を `View` に渡しつつ、実行処理を指示しています。  

##### Viewの説明
`ViewModel` に続いて `View` を説明します。  
繰り返しになりますが、 `View` の責務は、  

* `View`  
  * ユーザアクションのキャッチ  
  * 画面の描画  

になります。  
まずは、 `View` の最低限の実装から説明します。  

```objective-c
import UIKit
import MapKit
import RxSwift
import RxCocoa

// 説明(1)
final class MapViewController: UIViewController, Injectable {
    typealias Dependency = MapViewModel

    // MARK: - IBOutlets
    @IBOutlet private weak var mapView: MKMapView!
    @IBOutlet private weak var tabBar: UITabBar!
    @IBOutlet private weak var searchButton: UIButton!

    // MARK: - Properties
    // 説明(2)
    private let viewModel: MapViewModel
    private let disposeBag = DisposeBag()

    // MARK: - Initial methods
    // 説明(3)
    required init(with dependency: Dependency) {
        viewModel = dependency
        super.init(nibName: nil, bundle: nil)
    }

    @available(*, unavailable)
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    // MARK: - Lifecycle methods
    override func viewDidLoad() {
        super.viewDidLoad()
        ...
    }
}
```

**説明(1)**  
`ViewModel` と `View` を疎結合にするために、ここでも `Injectable` を利用します。  
`MapViewController` を `Injectable` プロトコルを継承するクラスとして定義し、  
`Dependency` を `MapViewModel` の別名として設定しています。  

**説明(2)**  
`ViewModel` からの指示を受け取れるように、 `MapViewModel` を宣言します。   

**説明(3)**  
`Injectable` プロトコルを継承するため、 `Dependency` を引数に持つ `init` メソッドが必要になります。  
メソッド内で `viewModel` に `dependency` を与えていますが、  
これは冒頭で説明した通り `Dependency` を `MapViewModel` の別名として設定しているため実現可能となります。  

最低限の実装の次は「位置情報の計測を停止する」機能を元に、 `View` → `ViewModel` の実装を見てみます。  

本アプリでは、`UITabBar`の「STOP」項目をタップすることで位置情報の計測停止処理を進めることができます。  
よって、 `View` → `ViewModel` の部分は下記のように...    

```objective-c
 // 説明(4)
 // Drive to ViewModel
 private func driveToViewModel() {
    tabBar.rx.didSelectItem
        .asDriver()
        .drive(onNext: { [weak self] item in
            guard let strongSelf = self else { return }
            strongSelf.didSelectTabBarItem(tag: item.tag)
            }, onCompleted: nil, onDisposed: nil)
        .disposed(by: disposeBag)
 }

// 説明(5)
/// 各タブバーアイテムタップ時の処理
///
/// - Parameter tag: タブバーアイテムのタグ
private func didSelectTabBarItem(tag: Int) {
    guard let itemTag = TabBarItemTag(rawValue: tag) else { return }
    switch itemTag {
    case .start:
        startUpdatingLocationMode()
    case .stop:
        stopUpdatingLocationMode()
    case .footView:
        showOrHideFootprintMode()
    case .settings:
        showSettingViewMode()
    }
}

// 説明(6)
/// Stopモードに変更された場合に実行される処理
private func stopUpdatingLocationMode() {
    // 確認アラートを表示、タブバーの選択表示をnilにする(全て未選択状態にする)
    let alert = UIAlertController(title: R.string.common.confirmTitle(),
                                  message: R.string.mapView.stopUpdatingLocationMessage(),
                                  preferredStyle: .alert)
    self.promptFor(alert: alert)
        .subscribe({ [weak self] event in
            // アラートを消す
            alert.dismiss(animated: false, completion: nil)

            // アラートに表示されたOK/Cancelボタンのどちらをタップしたか確認
            guard let strongSelf = self, let alertActionType = event.element else { return }
            switch alertActionType {
            case .ok:
                // OKボタンをタップした場合
                // タブバーの全アイテムを未選択の状態にする
                strongSelf.tabBar.selectedItem = nil
                // ストップボタンをdisabledに変更
                strongSelf.activateStartButton()
                // 位置情報の取得停止をViewModelにバインディング
                Observable.just(Void())
                    .bind(to: strongSelf.viewModel.stopUpdatingLocation)
                    .disposed(by: strongSelf.disposeBag)
            case .cancel:
                // Cancelボタンをタップした場合
                // タブバーの選択状態をスタートボタンの選択状態に戻す
                let startTag = TabBarItemTag.start
                strongSelf.tabBar.selectedItem = strongSelf.tabBar.items?[startTag.rawValue]
            }
        })
        .disposed(by: disposeBag)
}
```

**説明(4)**  
`RxCocoa` 内に `UITabBar+Rx.swift` があり、その中で `Rx` 的に扱えるように `didSelectItem` が定義されています。  
ここでは、それを用いて、 `UITabBar` のタブ項目をタップしたら、 `didSelectTabBarItem` メソッドを呼び出すように処理を書いています。  

**説明(5)**  
ここは1つ1つの処理が長くなり過ぎないように、単にメソッド分けしているだけです。  
`tag` の `0` 〜 `3` で判別して処理分けしているのですが、  
直で数字で `switch` 文を利用したくないので `TabBarItemTag` を定義しています。  

```objective-c
// TabBarItemTag.swift
enum TabBarItemTag: Int {
    case start = 0
    case stop
    case footView
    case settings
}
```

単にこれだけですが、何をタップした時にどんな処理をするのかが、こちらの方がひと目でわかりますよね。  

**説明(6)**  
ここで具体的に「STOP」をタップされた場合の処理を書いています。  
重要なのは、  

```objective-c
// 位置情報の取得停止をViewModelにバインディング
Observable.just(Void())
    .bind(to: strongSelf.viewModel.stopUpdatingLocation)
    .disposed(by: strongSelf.disposeBag)
```

の部分です。  
「位置情報の計測を停止しますか？」という質問に「OK」と答えた際に実行される処理で、  
`View` から `ViewModel` に指示が出ていることを伝えています。  
(`Void`型のデータを `viewModel.stopUpdatingLocation` にバインディングしています。)  

このような形で `View` と `ViewModel` は双方向データバインディングな関係を構築しています。  

#### 今後の展望とリリース時期について
ざっくりと `MVVM` で実装したソースコードを説明してきましたが、  
冒頭でも述べた通り、まだアプリは完成しておりません...  

そこで今後の展望ですが、下記2点となります。  

* `RxSwift` らしい書き方に修正する  
  * 学習しながら実装していた経緯もあり、無用に `PublishSubject` や `BehaviorRelay` を利用している箇所があります。  
  * 上記を `Driver` に置き換えることで視認性の向上に繋がると考えています。  
* テストの拡充
  * 正直、まだ十分にテストが書けていません...  
  * 折角、疎結合を意識しながら構築しているのでテストは書き切りたいと思っています。  

それらを満たした上で、リリース時期は1月末を見込んでいます。  
極力、早期なリリースを目指していきたいと思います。  

### まとめ
さて如何でしたでしょうか？  
今回は5年前の個人開発アプリの再構築について紹介させて頂きました。  

個人開発すると、業務で学んだ技術の復習になることもあるでしょうし、  
新たな技術の学びにも繋がるかと思います。  

筆者もめげずにこれからも新しい技術を学び続け、個人開発した結果をアウトプットしていきたいなと思います。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
