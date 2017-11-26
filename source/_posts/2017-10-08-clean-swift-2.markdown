---
layout: post
title: "Clean Swiftを勉強してみよう！(2)"
date: 2017-10-08 22:35
comments: true
categories: ios swift
---

### はじめに
今回は[Clean Swift](https://clean-swift.com/)を用いた具体的な例について見ていきたいと思います。  
題材として、下記のような要件を持つアプリを扱います。  

![Clean Swiftの題材アプリ](/images/clean-swift-2_1.png)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### アプリの要件
要件としては、下記の通りです。  

* 現在地を中心にマップを表示する  
* 検索ボタンをタップして、現在地周辺のレストラン情報を取得する  
* マップに表示されたレストランのマーカをタップすると、レストラン情報の概要ウィンドウが表示される  

### フォルダ構成
フォルダ構成は以下のようになっています。  
※ 今回のアプリの名称を `CleanFoodLogger` とします。  

```javascript
CleanFoodLogger
├── Views
│    └── ViewController+Alert.swift
├── Models
│    ├── Restaurant.swift
│    └── CustomGMSMarker.swift
├── Workers
│    └── HotpepperWorker.swift
├── Services
│    └── HotpepperAPI.swift
├── Scenes
│    └── MapView
│         ├── View
│         │    ├── CustomInfoWindow.xib
│         │    └── CustomInfoWindow.swift
│         ├── MapViewController.swift
│         ├── MapInteractor.swift
│         ├── MapModels.swift
│         ├── MapPresenter.swift
│         ├── MapRouter.swift
│         └── MapWorker.swift
├── AppDelegate.swift
├── Main.storyboard
├── Assets.xcassets
└── key.plist
```

それぞれの構成の意味について説明します。  

#### Views
`Scene` に寄らない共通ビュー系を格納します。  

* `ViewController+Alert.swift`  
  * `UIViewController` を拡張する形で実装  
  * 共通アラート表示処理を実装  

#### Models
`Scene` に寄らないモデル系を格納します。  

* `Restaurant.swift`  
  * APIを通して取得したレストラン情報を格納するモデル  
* `CustomGMSMarker`  
  * `GMSMarker`を継承したカスタムクラス  
  * マーカにショップ情報を追加して持たせたモデル  

#### Workers
`Scene` に寄らない `Clean Swift` で言うところの `Worker` 系を格納します。  
※ 今回は、レストラン情報を取得するのに[ホットペッパーAPI](https://webservice.recruit.co.jp/hotpepper/reference.html)を利用しています。  

* `HotpepperWorker.swift`  
  * ホットペッパーAPI管理マネージャに当たる `Services/HotpepperAPI.swift` を通じて取得したレストラン情報を扱う  

#### Services
`Scene` に寄らない管理マネージャ系の共通処理を扱います。  

* `HotpepperAPI.swift`  
  * ホットペッパーAPIを用いた具体的なロジックを実装  

#### Scenes
ここは画面単位に `Clean Swift`のテンプレートを当て込んだ構成になります。  
今回は簡単なサンプルなので1画面しかありません。  
よって、 `Scenes` に格納しているのも `MapView` 1つになります。  
`MapView` の配下は `Clean Swift` テンプレート一式になっています。   

### アプリの実装
続いて具体的なアプリの実装について見ていきます。  
データフローとしては、下記の通りです。  

![データフロー図](/images/clean-swift_3.png)  

ではまずは、共通系の処理から説明します。  

#### Models
この後の `Services` や `Workers` にも出てくるので、まずは `Models` から説明します。  

##### Restaurant.swift
ホットペッパーAPIで取得したレストラン情報の一部を抜粋して格納するため、下記のような構成になっています。  
今回は `==` で比較する処理を利用している箇所はありませんが、 [Clean Store](https://github.com/Clean-Swift/CleanStore) を参考にしたので、そのまま残しています。  

```objective-c
import Foundation

struct Restaurant: Equatable {

    var id: String
    var name: String
    var category: String
    var imageURL: String
    var latitude: Double
    var longitude: Double
}

func ==(lhs: Restaurant, rhs: Restaurant) -> Bool {
    return lhs.id == rhs.id
        && lhs.name == rhs.name
        && lhs.category == rhs.category
        && lhs.imageURL == rhs.imageURL
        && lhs.latitude == rhs.latitude
        && lhs.longitude == rhs.longitude
}
```

##### CustomGMSMarker.swift
マーカをタップした際に、ショップ情報を表示する `InfoWindow` を表示する要件があるため、 `GMSMarker` クラスを継承して、ショップ情報を追加しています。  

```objective-c
import Foundation
import GoogleMaps

class CustomGMSMarker: GMSMarker {

    public var id: String!
    public var name: String!
    public var category: String!
    public var imageURL: String!

    /// 初期化
    override init() {
        super.init()
    }
}
```

#### Workers
`Services` の説明に移る前に、`Protocol` を提供している `Workers` について見ていきます。  

##### HotpepperWorker.swift
これは、  

* API処理を扱うための `Worker` クラスを提供  
* 外部へのインターフェースを定義したプロトコルの定義  

の役目を果たしています。  

```objective-c
import Foundation

// MARK: - Hotpepper worker

class HotpepperWorker {
    var hotpepper: HotpepperProtocol

    init(hotpepper: HotpepperProtocol) {
        self.hotpepper = hotpepper
    }

    func fetchRestaurants(latitude: Double, longitude: Double, completionHandler: @escaping ([Restaurant], HotpepperError?) -> Void) {
        hotpepper.fetchRestaurants(latitude: latitude, longitude: longitude) { (restaurants, error) in
            DispatchQueue.main.async {
                completionHandler(restaurants, error)
            }
        }
    }
}

// MARK: Hotpepper API

protocol HotpepperProtocol {

    // MARK: CRUD operations

    func fetchRestaurants(latitude: Double, longitude: Double, completionHandler: @escaping ([Restaurant], HotpepperError?) -> Void)
}

// MARK: - CRUD operation errors

enum HotpepperError: Equatable, Error {
    case CannotFetch(String)
}

func ==(lhs: HotpepperError, rhs: HotpepperError) -> Bool {
    switch (lhs, rhs) {
    case (.CannotFetch(let a), .CannotFetch(let b)) where a == b: return true
    default: return false
    }
}
```

#### Services
`Worker` で定義された外部へのインターフェースの挙動を実装したクラスに当たります。  

##### HotpepperAPI.swift
今回は `HotpepperProtocol` を継承した `HotpepperAPI` 内で実際にホットペッパーAPIを叩いて処理しています。  

```objective-c
import Foundation
import Alamofire
import SwiftyJSON

class HotpepperAPI: HotpepperProtocol {

    /// API Key
    private var apiKey: String = String()
    /// Geocoding APIのベースURL
    private let baseURL: String = "https://webservice.recruit.co.jp/hotpepper/gourmet/v1/"

    /// 初期化処理
    // key.plistに定義したAPIKeyを取得してセット
    init() {
        if let path = Bundle.main.path(forResource: "key", ofType: "plist") {
            if let dic = NSDictionary(contentsOfFile: path) as? [String: Any] {
                if let apiKey = dic["hotpepperApiKey"] as? String {
                    self.apiKey = apiKey
                }
            }
        }
    }

    // HotpepperWorker.swift内のHotpepperProtocolインターフェースの具体的な処理
    func fetchRestaurants(latitude: Double, longitude: Double, completionHandler: @escaping ([Restaurant], HotpepperError?) -> Void) {
        let parameters = ["key": self.apiKey, "format": "json", "lat": latitude, "lng": longitude, "range": 3] as [String : Any]
        Alamofire.SessionManager.default.requestWithoutCache(baseURL, method: .get, parameters: parameters, encoding: URLEncoding.default, headers: nil).responseJSON { response in
            var restaurants: [Restaurant] = [Restaurant]()

            if response.result.isFailure {
                let defaultErrorMessage = "レストラン情報を取得できませんでした。"
                completionHandler([], HotpepperError.CannotFetch(response.result.error?.localizedDescription ?? defaultErrorMessage))
                return
            }

            let json = JSON(response.result.value as Any)
            guard let shops = json["results"]["shop"].array else {
                let defaultErrorMessage = "レストラン情報を取得できませんでした。"
                completionHandler([], HotpepperError.CannotFetch(response.result.error?.localizedDescription ?? defaultErrorMessage))
                return
            }

            for shop in shops {
                let id = shop["id"].string ?? "ID不明"
                let name = shop["name"].string ?? "ショップ名不明"
                let category = shop["genre"]["name"].string ?? "カテゴリ不明"
                let imageURL = shop["photo"]["mobile"]["l"].string ?? ""
                let latitude = atof(shop["lat"].string ?? "0")
                let longitude = atof(shop["lng"].string ?? "0")
                let restaurant = Restaurant(id: id, name: name, category: category, imageURL: imageURL, latitude: latitude, longitude: longitude)
                restaurants.append(restaurant)
            }

            completionHandler(restaurants, nil)
        }
    }
}

// Clean Swiftとは無関係ですが、キャッシュなしリクエストをAlamofireを通して実装する処理
extension Alamofire.SessionManager {
    @discardableResult
    open func requestWithoutCache(
        _ url: URLConvertible,
        method: HTTPMethod = .get,
        parameters: Parameters? = nil,
        encoding: ParameterEncoding = URLEncoding.default,
        headers: HTTPHeaders? = nil)
        -> DataRequest {
            do {
                var urlRequest = try URLRequest(url: url, method: method, headers: headers)
                urlRequest.cachePolicy = .reloadIgnoringCacheData // <<== Cache disabled
                let encodedURLRequest = try encoding.encode(urlRequest, with: parameters)
                return request(encodedURLRequest)
            } catch {
                print(error)
                return request(URLRequest(url: URL(string: "http://example.com/wrong_request")!))
            }
    }
}
```

#### MapView
ここから重要な `Clean Swift` を使った実装に入っていきます。  
今回は共通 `Worker` のみ利用しているため、 `MapWorker.swift` は省略します。  
また、画面遷移の処理もないため、 `MapRouter.swift` についても省略します。  

##### MapModels.swift
Clean Swiftで今回扱う `Model` は以下の通りです。  

```objective-c
import UIKit

enum Map {

    // MARK: Init mapView

    enum Init {
        struct Request {
            var latitude: Double
            var longitude: Double
        }
        struct Response {
            var latitude: Double
            var longitude: Double
        }
        struct ViewModel {
            var latitude: Double
            var longitude: Double
            var zoomLevel: Float
        }
    }

    // MARK: Search restaurants

    enum Search {
        struct Request {
            var latitude: Double
            var longitude: Double
        }
        struct Response {
            var restaurants: [Restaurant]
        }
        struct ViewModel {
            var restaurants: [Restaurant]
        }
    }
}
```

* `Init`  
  * マップ画面の初期描画時に「現在地を中心としたマップ位置に移動する」際に利用  
* `Search`  
  * 「現在地周辺のレストランを検索する」際に利用

##### MapInteractor.swift
`ViewController` から受け取った依頼を `Worker` を経由して取得した値を `Presenter` に渡します。  

```objective-c
import UIKit

protocol MapBusinessLogic {
    func initMapView(request: Map.Init.Request)
    func searchRestaurants(request: Map.Search.Request)
}

protocol MapDataStore {
}

class MapInteractor: MapBusinessLogic, MapDataStore {
    var presenter: MapPresentationLogic?
    var worker = HotpepperWorker(hotpepper: HotpepperAPI())
    private var initView: Bool = false

    // MARK: Init mapView

    func initMapView(request: Map.Init.Request) {
        if !initView {
            let response = Map.Init.Response(latitude: request.latitude, longitude: request.longitude)
            presenter?.presentInitMapView(response: response)
            initView = true
        }
    }

    // MARK: Search restaurants

    func searchRestaurants(request: Map.Search.Request) {
        worker.fetchRestaurants(latitude: request.latitude, longitude: request.longitude) { (restaurants, error) in
            let response = Map.Search.Response(restaurants: restaurants)
            self.presenter?.presentSearchedRestaurants(response: response)
        }
    }
}
```

* `initMapView`  
  * APIやローカルDBを利用する必要がないため、`Worker`は利用していません  
  * 初回だけ、実行すれば良い処理なので内部で定義した `initView` でハンドリングしています  
* `searchRestaurants`  
  * ホットペッパーAPIによるデータ取得は `HotpepperWorker` に任せています  
  * `HotpepperWorker` を介して取得したデータを `Map.Search.Response` 形式に変換  
  * それを `MapPresenter` に渡しています  

##### MapPresenter.swift
`Interactor` から受け取ったデータを表示形式に変換して、`ViewController` に描画指示を出します。  

```objective-c
import UIKit

protocol MapPresentationLogic {
    func presentInitMapView(response: Map.Init.Response)
    func presentSearchedRestaurants(response: Map.Search.Response)
}

class MapPresenter: MapPresentationLogic {
    weak var viewController: MapDisplayLogic?
    private let zoomLevel: Float = 16.0

    // MARK: Present init mapView

    func presentInitMapView(response: Map.Init.Response) {
        let latitude = response.latitude
        let longitude = response.longitude
        let viewModel = Map.Init.ViewModel(latitude: latitude, longitude: longitude, zoomLevel: zoomLevel)
        viewController?.displayInitMap(viewModel: viewModel)
    }

    // MARK: Present searched restaurants

    func presentSearchedRestaurants(response: Map.Search.Response) {
        let restaurants = response.restaurants
        let viewModel = Map.Search.ViewModel(restaurants: restaurants)
        if restaurants.count > 0 {
            viewController?.displaySearchedSuccess(viewModel: viewModel)
            return
        }
        viewController?.displaySearchedFailure(viewModel: viewModel)
    }
}
```

* `presentInitMapView`  
  * 「現在地を中心としたマップ位置に移動する」ために `ViewController` に緯度/経度/ズームレベルを渡します  
* `presentSearchedRestaurants`    
  * レストラン情報の有無で `ViewController` に出す指示を変えています  
  * 今回はシンプルな実装のため、`Map.Search.Response` から `Map.Search.ViewModel` に変換はありません  

##### MapViewController.swift
最後に `ViewController` について説明します。  
下記で  

* `Interactor` に具体的な処理内容(表示ロジック)を問い合わせる
* `Presenter` からの指示を受けて、最適な `View` を描画する

を実現しています。  

```objective-c
import UIKit
import GoogleMaps

protocol MapDisplayLogic: class {
    func displayInitMap(viewModel: Map.Init.ViewModel)
    func displaySearchedSuccess(viewModel: Map.Search.ViewModel)
    func displaySearchedFailure(viewModel: Map.Search.ViewModel)
}

class MapViewController: UIViewController, MapDisplayLogic {
    var interactor: MapBusinessLogic?
    var router: (NSObjectProtocol & MapRoutingLogic & MapDataPassing)?

    @IBOutlet weak var mapView: GMSMapView!
    var locationManager: CLLocationManager?

    // MARK: Object lifecycle

    override init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: Bundle?) {
        super.init(nibName: nibNameOrNil, bundle: nibBundleOrNil)
        setup()
    }

    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
        setup()
    }

    // MARK: Setup

    private func setup() {
        let viewController = self
        let interactor = MapInteractor()
        let presenter = MapPresenter()
        let router = MapRouter()
        viewController.interactor = interactor
        viewController.router = router
        interactor.presenter = presenter
        presenter.viewController = viewController
        router.viewController = viewController
        router.dataStore = interactor
    }

    // MARK: Routing

    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        if let scene = segue.identifier {
            let selector = NSSelectorFromString("routeTo\(scene)WithSegue:")
            if let router = router, router.responds(to: selector) {
                router.perform(selector, with: segue)
            }
        }
    }

    // MARK: View lifecycle

    override func viewDidLoad() {
        super.viewDidLoad()

        configureMapView()
        configureLocationManager()
    }

    // MARK: Configuration
    func configureMapView() {
        // GoogleMapの初期化
        mapView.isMyLocationEnabled = true
        mapView.mapType = GMSMapViewType.normal
        mapView.settings.compassButton = true
        mapView.settings.myLocationButton = true
        mapView.settings.compassButton = true
        mapView.delegate = self
    }

    func configureLocationManager() {
        // 位置情報関連の初期化
        self.locationManager = CLLocationManager()
        self.locationManager?.desiredAccuracy = kCLLocationAccuracyBest
        self.locationManager?.requestWhenInUseAuthorization()
        self.locationManager?.startUpdatingLocation()
        self.locationManager?.delegate = self
    }

    // MARK: Init mapView

    func displayInitMap(viewModel: Map.Init.ViewModel) {
        // 初期描画時のマップ中心位置の移動
        let coordinate = CLLocationCoordinate2D(latitude: viewModel.latitude, longitude: viewModel.longitude)
        let camera = GMSCameraPosition.camera(withTarget: coordinate, zoom: viewModel.zoomLevel)
        mapView.camera = camera
    }

    // MARK: Search restaurants

    func searchRestaurants() {
        guard let latitude = mapView.myLocation?.coordinate.latitude, let longitude = mapView.myLocation?.coordinate.longitude else {
            return
        }
        let request = Map.Search.Request(latitude: latitude, longitude: longitude)
        interactor?.searchRestaurants(request: request)
    }

    func displaySearchedSuccess(viewModel: Map.Search.ViewModel) {
        let restaurants = viewModel.restaurants
        for restaurant in restaurants {
            putMarker(restaurant: restaurant)
        }
    }

    func displaySearchedFailure(viewModel: Map.Search.ViewModel) {
        showAlert(title: "確認", message: "周辺にレストランは見つかりませんでした。") {
        }
    }

    @IBAction func tappedSearchButton(_ sender: Any) {
        searchRestaurants()
    }

    // MARK: Other
    private func putMarker(restaurant: Restaurant) {
        let marker = CustomGMSMarker()
        marker.id = restaurant.id
        marker.name = restaurant.name
        marker.category = restaurant.category
        marker.imageURL = restaurant.imageURL
        marker.position = CLLocationCoordinate2D(latitude: restaurant.latitude, longitude: restaurant.longitude)
        marker.icon = UIImage(named: "RestaurantIcon")
        marker.appearAnimation = GMSMarkerAnimation.pop
        marker.map = mapView
    }
}

extension MapViewController: CLLocationManagerDelegate {

    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        // マップの初期描画
        if let coordinate = locations.last?.coordinate {
            let request = Map.Init.Request(latitude: coordinate.latitude, longitude: coordinate.longitude)
            interactor?.initMapView(request: request)
        }
    }
}

extension MapViewController: GMSMapViewDelegate {

    func mapView(_ mapView: GMSMapView, markerInfoWindow marker: GMSMarker) -> UIView? {
        guard let cMarker = marker as? CustomGMSMarker else {
            return nil
        }

        cMarker.tracksInfoWindowChanges = true
        let view = CustomInfoWindow(frame: CGRect(x: 0, y: 0, width: 250, height: 265))
        view.setup(name: cMarker.name, category: cMarker.category, imageURL: cMarker.imageURL)

        return view
    }
}
```

* `configureMapView` / `configureLocationManager`  
  * 最低限必要な `ViewController` 上での設定処理  
  * ここで `startUpdatingLocation` を実行することで現在地の更新を開始  
* `didUpdateLocations` → `displayInitMap`  
  * 初期起動時は `mapView.myLocation` から現在地の即時取得ができないため、`startUpdatingLocation` を利用しています  
  * 現在地が取得できたタイミングで `didUpdateLocations` を通るため、 `Interactor` にマップ中心位置の移動を依頼しています  
  * 位置を移動させるか否かは `ViewController` では判断しません  
* `tappedSearchButton` → `searchRestaurants` → `displaySearchedSuccess` / `displaySearchedFailure`  
  * 検索ボタンをタップした時に `searchRestaurants` を呼び出しています  
  * `Interactor` にレストラン情報の検索を依頼しています  
  * `Presenter` から `displaySearchedSuccess` または `displaySearchedFailure` の描画指示を受けて描画します  
  * `Presenter` から返却された `Map.Search.ViewModel` を利用して `putMarker` を実行することでマップにマーカをプロットします。  
  * `displaySearchedFailure` では、失敗したことをアラート表示することで表現しています   
* `GMSMapViewDelegate`  
  * マーカタップ時の処理を実装しています  

### まとめ
さて如何でしたでしょうか？  
次回は今回扱ったサンプルを拡張する形で実装し、説明していきたいと思います。  
因みに、本記事のソースは [CleanFoodLogger](https://github.com/grandbig/CleanFoodLogger)にて公開しています。  
※ バージョン `1.0` を参照してください。  

ということで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
