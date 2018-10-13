---
layout: post
title: "GeolocationSampleから学ぶdelegateのRx対応"
date: 2018-10-06 11:16
comments: true
categories: ios swift rx
---

### はじめに
`RxSwift` を利用して `MVVM` アーキテクチャでアプリを開発することがあるでしょう。  
その際に、ボタンタップやネットワーク通信であれば、何もやらずとも `RxSwift` が対応してくれていたり、 `RxSwift` に対応しているライブラリがあったりします。  

しかし、デフォルトでは `RxSwift` に対応していない場合も当然あります。  
ではそんなとき、どのようにして対応すれば良いでしょうか。  

今日は、 `delegate` の `Rx` 対応について公式サンプルの `GeolocationSample` を元に説明してみたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### delegateのRx対応方法
早速具体的に方法を見ていきましょう。  
今回は公式サンプルの `GeolocationSample` を元に、 `CLLocationManagerDelegate` を `Rx` に対応させる方法を説明します。  

#### DelegateProxyとDelegateProxyTypeへの対応
`delegate` の `Rx` 対応でまず必要なことは  

* `DelegateProxy` クラスを継承するクラスを作成すること  
* `DelegateProxyType` プロトコルを継承するクラスを作成すること  

です。  
ここでは上記2つの条件を満たした `RxCLLocationManagerDelegateProxy` クラスを作ることとします。  

##### DelegateProxyの説明

`DelegateProxy.swift` を見てみると、下記のように定義されています。  

```objective-c
// DelegateProxy.swift

/// Base class for `DelegateProxyType` protocol.
///
/// This implementation is not thread safe and can be used only from one thread (Main thread).
open class DelegateProxy<P: AnyObject, D>: _RXDelegateProxy {
    public typealias ParentObject = P
    public typealias Delegate = D
    ...
}
```

この `DelegateProxy` は `DelegateProxyType` プロトコルのベースクラスと説明されています。  
`DelegateProxy` はジェネリッククラスであり、2つのパラメータ `P` と `D` を持ちます。  

ここで、 `P` と `D` について説明します。  

* `D` :  
`Rx` に対応させたい `delegate` を指定します  
`D` は `Delegate` の頭文字と思われます  
* `P` :  
`delegate` である `D` をプロパティとして持つオブジェクトを指定します  
`P` は `ParentObject` の頭文字と思われます  

今回の場合は、  
`DelegateProxy<CLLocationManager, CLLocationManagerDelegate>` になります。  

##### DelegateProxyTypeの説明

`DelegateProxyType.swift` の中身を見てみると、下記のように説明されています。  

```objective-c
// DelegateProxyType.swift

/**
`DelegateProxyType` protocol enables using both normal delegates and Rx observable sequences with
views that can have only one delegate/datasource registered.
...

*/
```

意訳すると、  
`DelegateProxyType` は `delegate` と `Rx` との紐付けを実現するプロトコル  
であることを指しています。  

方式は図示化されていますので、見てみると何となく理解できると思います。  
図では `UIScrollViewDelegate` を例に説明されています。  

```objective-c
+-------------------------------------------+
|                                           |                           
| UIView subclass (UIScrollView)            |                           
|                                           |
+-----------+-------------------------------+                           
            |                                                           
            | Delegate                                                  
            |                                                           
            |                                                           
+-----------v-------------------------------+                           
|                                           |                           
| Delegate proxy : DelegateProxyType        +-----+---->  Observable<T1>
|                , UIScrollViewDelegate     |     |
+-----------+-------------------------------+     +---->  Observable<T2>
            |                                     |                     
            |                                     +---->  Observable<T3>
            |                                     |                     
            | forwards events                     |
            | to custom delegate                  |
            |                                     v                     
+-----------v-------------------------------+                           
|                                           |                           
| Custom delegate (UIScrollViewDelegate)    |                           
|                                           |
+-------------------------------------------+   
```

また `DelegateProxyType` は以下3つの `static` メソッドを定義しているため、  
`DelegateProxyType` を継承すると、必ずこの3つのメソッドを持つ必要があります。  

* `registerKnownImplementations`  
このメソッドの中で必ず `DelegateProxySubclass.register()` を実行します。  
これをすることで自身で定義した `DelegateProxy` の継承クラスを登録することができます。  
* `currentDelegate`  
`ParentObject` の持つ `delegate` を返却する処理を書きます。  
* `setCurrentDelegate`  
`ParentObject` に持つべき `delegate` を設定する処理を書きます。  

特に特殊なことをしない場合は、  
`delegate` をプロパティとして持つオブジェクトである `ParentObject`に  
`HasDelegate` プロトコルを継承させます。 　

```objective-c
extension CLLocationManager: HasDelegate {
    public typealias Delegate = CLLocationManagerDelegate
}
```

これにより、 `currentDelegate` と `setCurrentDelegate` を省略することができます。  

##### 対応したコードを書いてみる
基本的な説明は以上として、実際にコードに起こしてみましょう。  
まずは結果から。  

```objective-c
import RxSwift      // ここは必須
import RxCocoa      // ここは必須
import CoreLocation //  CLLocationManagerDelegateはCoreLocation内に定義されています

// currentDelegateとsetCurrentDelegateの役割を担います
extension CLLocationManager: HasDelegate {
    public typealias Delegate = CLLocationManagerDelegate
}

// DelegateProxy, DelegateProxyType, CLLocationManagerDelegateを継承
// DelegateをRxに対応させるために、元となるDelegateも継承が必須です
public class RxCLLocationManagerDelegateProxy: DelegateProxy<CLLocationManager, CLLocationManagerDelegate>,
    DelegateProxyType,
    CLLocationManagerDelegate {

    // 初期化処理
    public init(locationManager: CLLocationManager) {
        super.init(parentObject: locationManager, delegateProxy: RxCLLocationManagerDelegateProxy.self)
    }

    // 必須のstaticメソッド
    public static func registerKnownImplementations() {
        // 説明(1)
        self.register { (locationManager) -> RxCLLocationManagerDelegateProxy in
            RxCLLocationManagerDelegateProxy(locationManager: locationManager)
        }
    }

    // 説明(2)
    internal lazy var didUpdateLocationsSubject = PublishSubject<[CLLocation]>()
    internal lazy var didFailWithErrorSubject = PublishSubject<Error>()

    // 説明(3)
    public func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        _forwardToDelegate?.locationManager(manager, didUpdateLocations: locations)
        didUpdateLocationsSubject.onNext(locations)
    }

    public func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        _forwardToDelegate?.locationManager(manager, didFailWithError: error)
        didFailWithErrorSubject.onNext(error)
    }

    // 説明(4)
    deinit {
        self.didUpdateLocationsSubject.on(.completed)
        self.didFailWithErrorSubject.on(.completed)
    }
}
```

上記ソースコードを一部補足説明します。  

###### 説明(1)
`registerKnownImplementations` で説明した通り `register` メソッドを実行しています。  
`register` メソッドは、   

```objective-c
/// Store DelegateProxy subclass to factory.
/// When make 'Rx*DelegateProxy' subclass, call 'Rx*DelegateProxySubclass.register(for:_)' 1 time, or use it in DelegateProxyFactory
/// 'Rx*DelegateProxy' can have one subclass implementation per concrete ParentObject type.
/// Should call it from concrete DelegateProxy type, not generic.
public static func register<Parent>(make: @escaping (Parent) -> Self) {
    self.factory.extend(make: make)
}
```

と定義されています。  

クロージャの引数に `ParentObject` を必要とし、  
そのクラス自身を戻り値を必要としているため、  
`ParentObject` として `locationManager` を渡し、  
それを元に初期化した `RxCLLocationManagerDelegateProxy` オブジェクトを戻り値として渡しています。   

ここは説明のため省略書きしませんでしたが、  

```objective-c
public static func registerKnownImplementations() {
    self.register { RxCLLocationManagerDelegateProxy(locationManager: $0) }
}
```

とも当然書けます。  

###### 説明(2)
`PublishSubject` 型のプロパティを2つ定義しています。  

```objective-c
internal lazy var didUpdateLocationsSubject = PublishSubject<[CLLocation]>()
internal lazy var didFailWithErrorSubject = PublishSubject<Error>()
```

これは説明(3)にも関わるのですが、  
`delegate` メソッドが呼び出されて処理が実行されたことを `Subscriber` に伝えるために定義が必要となります。  

###### 説明(3)
`delegate` メソッドを `Rx` で対応するための方法が、まさにココで直接的に書かれています。  

```objective-c
public func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
    _forwardToDelegate?.locationManager(manager, didUpdateLocations: locations)
    didUpdateLocationsSubject.onNext(locations)
}
```

今回は、 `didUpdateLocations` で取得した `locations` の情報を `Rx` 連携させるために、上記のように記述しています。  
先程説明した `PublishSubject` が `Subscriber` にメソッドの実行タイミングでデータを伝える方法ですが、  
`didUpdateLocationsSubject.onNext(locations)` で実行しています。  

`_forwardToDelegate?.locationManager(manager, didUpdateLocations: locations)` はメモリ観点から  
`delegate` を引き続き利用していることを伝えるために利用しているように見えます。  

###### 説明(4)
最後に `deinit` 内で実行している処理ですが、  
`deinit` が呼ばれるということは初期化したオブジェクトが破棄される時なので、イベントが送られることはないはずです。  
よって `PublishSubject` からイベント送信完了を知らせるように実装しましょう。  

#### ReactiveへのCLLocationManagerの適応
事前準備が整ったため、実際に `CLLocationManager` を `Rx` 適応させてみます。  

`RxSwift` では下記のように書くことで拡張できる仕組みを用意しています。  

```objective-c
// CLLocationManager+Rx.swift
import CoreLocation
import RxSwift
import RxCocoa

extension Reactive where Base: CLLocationManager {
    ...
}
```

これが可能な理由は `Reactive.swift` を見てみると良いでしょう。  

```objective-c
public struct Reactive<Base> {
    /// Base object to extend.
    public let base: Base

    /// Creates extensions with base object.
    ///
    /// - parameter base: Base object.
    public init(_ base: Base) {
        self.base = base
    }
}
```

そして、拡張した後にやることは下記です。  

* `delegate` のラッパーを生成する  
* 各 `delegate` メソッドに対応したラッパープロパティを生成する  
* キャストメソッドを用意する  

1つずつ説明していきましょう。  

##### delegateのラッパーを生成する  
このラッパーは `delegate` を `DelegateProxy` 型として定義します。  
この `delegate` はもちろん `readOnly` で値の取得のみできるものとします。  
`DelegateProxy` の取得は `DelegateProxyType` プロトコルの `proxy` メソッドを利用します。  

```objective-c
/**
Reactive wrapper for `delegate`.

For more information take a look at `DelegateProxyType` protocol documentation.
*/
public var delegate: DelegateProxy<CLLocationManager, CLLocationManagerDelegate> {
    return RxCLLocationManagerDelegateProxy.proxy(for: base)
}
```

##### 各delegateメソッドに対応したラッパープロパティを生成する
`RxCLLocationManagerDelegateProxy` で `didUpdateLocations` と `didFailWithError` の `delegate` メソッドに対応しました。  
これらのメソッドに対応したラッパープロパティは以下のように実装します。  

```objective-c
// MARK: Responding to Location Events

/**
 Reactive wrapper for `delegate` message.
 */
 public var didUpdateLocations: Observable<[CLLocation]> {
    return RxCLLocationManagerDelegateProxy.proxy(for: base).didUpdateLocationsSubject.asObservable()
}

/**
 Reactive wrapper for `delegate` message.
 */
public var didFailWithError: Observable<Error> {
    return RxCLLocationManagerDelegateProxy.proxy(for: base).didFailWithErrorSubject.asObservable()
}
```

これらも `readOnly` で値のみを `Observable` 型で取得できるように定義しています。   

##### キャストメソッドを用意する
キャストメソッドを用意する理由は、  
あるメソッドの処理の完了タイミングで何らかの処理を実行させたい  
`methodInvoked` を利用するときに必要になります。  

処理は下記の通りです。  
`Optional` 型の場合とそうでない場合が必要になる可能性がありますので、2種類用意しています。   

```objective-c
fileprivate func castOrThrow<T>(_ resultType: T.Type, _ object: Any) throws -> T {
    guard let returnValue = object as? T else {
        throw RxCocoaError.castingError(object: object, targetType: resultType)
    }

    return returnValue
}

fileprivate func castOptionalOrThrow<T>(_ resultType: T.Type, _ object: Any) throws -> T? {
    if NSNull().isEqual(object) {
        return nil
    }

    guard let returnValue = object as? T else {
        throw RxCocoaError.castingError(object: object, targetType: resultType)
    }

    return returnValue
}
```

今回の場合、端末の位置情報を利用するので、 `CLLocationManagerDelegate` の `didChangeAuthorization` のハンドリングが必須になります。  
この `delegate` メソッドは定期的に繰り返し利用する必要はありません。  
状態が変わって、その情報を必要となったタイミングでだけ利用できれば良いのです。  
よって `methodInvoked` を利用してプロパティを定義します。  

```objective-c
// MARK: Responding to Authorization Changes

/**
 Reactive wrapper for `delegate` message.
 */
public var didChangeAuthorizationStatus: Observable<CLAuthorizationStatus> {
    return delegate.methodInvoked(#selector(CLLocationManagerDelegate.locationManager(_:didChangeAuthorization:)))
        .map { a in
            let number = try castOrThrow(NSNumber.self, a[1])
            return CLAuthorizationStatus(rawValue: Int32(number.intValue)) ?? .notDetermined
    }
}
```

以上で必要な対応は全て完了です。  

### Rxに対応したdelegateの使い方
自作した `Rx` 対応後の `delegate` を利用する例も見ていきましょう。  

#### 処理ロジックの実装
公式サンプルでは処理ロジックに相当する `GeolocationService.swift` を下記のように実装しています。  

```objective-c
// GeolocationService.swift
import CoreLocation
import RxSwift
import RxCocoa

class GeolocationService {

    static let instance = GeolocationService()
    // 説明(1)
    private (set) var authorized: Driver<Bool>
    private (set) var location: Driver<CLLocationCoordinate2D>

    private let locationManager = CLLocationManager()

    private init() {

        locationManager.distanceFilter = kCLDistanceFilterNone
        locationManager.desiredAccuracy = kCLLocationAccuracyBestForNavigation

        // 説明(2)
        authorized = Observable.deferred { [weak locationManager] in
                let status = CLLocationManager.authorizationStatus()
                guard let locationManager = locationManager else {
                    return Observable.just(status)
                }
                return locationManager
                    .rx.didChangeAuthorizationStatus
                    .startWith(status)
            }
            .asDriver(onErrorJustReturn: CLAuthorizationStatus.notDetermined)
            .map {
                switch $0 {
                case .authorizedAlways:
                    return true
                default:
                    return false
                }
            }

        // 説明(3)
        location = locationManager.rx.didUpdateLocations
            .asDriver(onErrorJustReturn: [])
            .flatMap {
                return $0.last.map(Driver.just) ?? Driver.empty()
            }
            .map { $0.coordinate }

        locationManager.requestAlwaysAuthorization()
        locationManager.startUpdatingLocation()
    }
}
```

1つずつ説明していきましょう。  

##### 説明(1)
今回のサンプルは、  

* 位置情報の利用を許可したら、画面が切り替わる  
* 取得した最新の位置情報を画面に表示する  

という、データ結果を画面に直接反映させる処理が含まれています。  
よって、  

```objective-c
private (set) var authorized: Driver<Bool>
private (set) var location: Driver<CLLocationCoordinate2D>
```

のように `Driver` として定義しています。  

##### 説明(2)
`authorized` は `delegate` メソッドである `didChangeAuthorization` が呼び出されたタイミングで値が変更される必要があります。   
今回は、  
`Subscribe` するまでは `Observable` を生成せずに、 `Subscribe` されたタイミングで `Observable` を返す `Observable` を生成する  
`deferred` メソッドを利用しています。  

```objective-c
authorized = Observable.deferred { [weak locationManager] in
        let status = CLLocationManager.authorizationStatus()
        guard let locationManager = locationManager else {
            return Observable.just(status)
        }
        // didChangeAuthorizationStatusからauthorizedの値を取得
        return locationManager
            .rx.didChangeAuthorizationStatus
            .startWith(status)
    }
    // エラーが発生した場合は .notDetermined で返却する
    .asDriver(onErrorJustReturn: CLAuthorizationStatus.notDetermined)
    .map {
        // .authorizedAlwaysの場合のみauthorizedにtrueを格納する
        switch $0 {
        case .authorizedAlways:
            return true
        default:
            return false
        }
    }
```

##### 説明(3)
最新の位置情報を取得したタイミングで通知します。  

```objective-c
location = locationManager.rx.didUpdateLocations
    // エラーが発生した場合は、空配列で返却する
    .asDriver(onErrorJustReturn: [])
    // 位置情報が格納されている場合はその値を、位置情報がない場合は空を返却する
    .flatMap {
        return $0.last.map(Driver.just) ?? Driver.empty()
    }
    // CLLocationCoordinate2Dの値を返却する
    .map { $0.coordinate }
```

#### Viewロジックの実装
サンプルでは下記のように実装しています。  

```objective-c
// GeolocationViewController.swift
import UIKit
import CoreLocation
import RxSwift
import RxCocoa

// 説明(1)
private extension Reactive where Base: UILabel {
    var coordinates: Binder<CLLocationCoordinate2D> {
        return Binder(base) { label, location in
            label.text = "Lat: \(location.latitude)\nLon: \(location.longitude)"
        }
    }
}

class GeolocationViewController: ViewController {

    @IBOutlet weak private var noGeolocationView: UIView!
    @IBOutlet weak private var button: UIButton!
    @IBOutlet weak private var button2: UIButton!
    @IBOutlet weak var label: UILabel!

    override func viewDidLoad() {
        super.viewDidLoad()

        view.addSubview(noGeolocationView)

        let geolocationService = GeolocationService.instance

        // 説明(2)
        geolocationService.authorized
            .drive(noGeolocationView.rx.isHidden)
            .disposed(by: disposeBag)

        // 説明(3)
        geolocationService.location
            .drive(label.rx.coordinates)
            .disposed(by: disposeBag)
        ...
    }
    ...
}
```

1つずつ説明していきましょう。  

##### 説明(1)
画面に位置情報を表示するために `UILabel` を独自に `Rx` に対応させています。  
これは `CLLocationManager` を拡張した方法と同じですね。  

##### 説明(2)
`authorized` が `true` の場合に `noGeolocationView` を非表示にするよう実装しています。   

##### 説明(3)
取得できた最新の位置情報を説明(1)で拡張した機能を利用して `UILabel` に表示するようにしています。  

以上で `Rx` に対応させた `delegate` を利用することができました。  

### まとめ
さて、如何でしたでしょうか。  
形式に沿って実装をすることで簡単に拡張することはできますが、  
実装1つ1つを理解することでより深く `RxSwift` を現場で活用できるかと思います。    

まだまだ筆者も理解が乏しいところがあるので、もっと深く勉強を続けていきたいと思います。  
ということで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
