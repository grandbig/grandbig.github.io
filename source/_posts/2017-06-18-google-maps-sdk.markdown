---
layout: post
title: "Google Maps SDK for iOSを導入してみよう！"
date: 2017-06-18 19:47
comments: true
categories: ios swift map google
---

### はじめに
本日は[Google Maps SDK for iOS](https://developers.google.com/maps/documentation/ios-sdk/?hl=ja)の導入の仕方を書きたいと思います。  
([3年以上前](https://grandbig.github.io/blog/2014/01/27/googlemapssdk2/)に遊んでいたようですが、全く記憶にない...)  

基本的には、[本家のスタートガイド](https://developers.google.com/maps/documentation/ios-sdk/start?hl=ja)に従って進めるだけで特に問題なく`Google Map`を実装できるでしょう。  
筆者の場合は`Storyboard`を使いたかったので少し気をつける必要がありました。  
では早速見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### Google Maps SDK for iOSの導入
実際に手順を書いていきます。  

**１．Xcodeでプロジェクトを作成します。**  
**２．`Podfile`を作成します。**  

```objective-c
# Podfile
use_frameworks!

target "GoogleMapsSample" do
  # Normal libraries
  pod 'GoogleMaps'
  pod 'GooglePlaces'

  abstract_target 'Tests' do
    inherit! :search_paths
    target "GoogleMapsSampleTests"
    target "GoogleMapsSampleUITests"

    pod 'Quick'
    pod 'Nimble'
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

**３．`pod install`で必要な`Framework`をインストールします。**  
**４．APIキーを取得します。**  
スタートガイドに従って下記ボタンをクリックすればAPIキーを取得することができます。  

![APIキーの取得](/images/google-maps-sdk-1.png)  

**５．アプリのAPIキーを読み取らせる処理を導入します。**  
筆者の場合、基本的なソースコードは`GitHub`に公開したかったため、そのままAPIキーをべた書きするわけにいきませんでした。  
なので、`key.plist`ファイルを作成し、このファイルを`GitHub`にアップしないという手法を取ることにしました。  

下図のように`key.plist`ファイルを作成  

![key.plistの作成](/images/google-maps-sdk-2.png)  

また下図のようなフォルダ構成で`Info.plist`と同じ階層に`key.plist`を配置しています。  
![フォルダ構成](/images/google-maps-sdk-6.png)  

`AppDelegate.swift`で`key.plist`からAPIキーを読み込みます。  

```objective-c
// AppDelegate.swift
import UIKit
import GoogleMaps
import GooglePlaces

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

  var window: UIWindow?

  func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    // Override point for customization after application launch.

    if let path = Bundle.main.path(forResource: "key", ofType: "plist") {
      if let dic = NSDictionary(contentsOfFile: path) as? [String: Any] {
        if let apiKey = dic["googleMapsApiKey"] as? String {
          GMSServices.provideAPIKey(apiKey)
          GMSPlacesClient.provideAPIKey(apiKey)
        }
      }
    }

    return true
  }
    ...
}
```

**６．`Storyboard`に`Google Map`を追加します。**  
本家スタートガイドだと`Storyboard`を利用しない方式での説明が書かれていましたが、筆者は`Storyboard`を利用しました。  

![StoryboardにGoogle Mapを配置](/images/google-maps-sdk-3.png)  

これで導入が完了です。  

### Google Cloud PlatformのAPI Managerで詳細設定
先程、本家のスタートガイドからAPIキーを取得していましたが、[Google Cloud Platform](https://console.cloud.google.com/home)内の`API Manager`からAPIキーに対して詳細設定をすることができます。  
例えば、スタートガイドから取得しただけでは、APIキーの用途が無制限になっています。  
この状態では意図しない大量利用に繋がる可能性もあるため、iOSアプリでのみ利用するなど詳細を設定した方が良いでしょう。  

`API Manager`ページには下記のように遷移できます。  
![Google Cloud Platform](/images/google-maps-sdk-4.png)    

APIキーの詳細設定は下記の通りです。  
![APIキーの詳細設定](/images/google-maps-sdk-5.png)  

### Google Mapで初期描画時の中心位置を現在地にしてみる
Apple標準で用意されている`MapKit`であれば、  

```objective-c
self.mapView.setUserTrackingMode(MKUserTrackingMode.follow, animated: true)
```

とすれば良いだけでした。  
しかし、`Google Map`では同様の手立てがなさそうなので、下記のように対応しました。  

```objective-c
// ViewController.swift
import UIKit
import GoogleMaps
import GooglePlaces

class ViewController: UIViewController, GMSMapViewDelegate, CLLocationManagerDelegate {

  @IBOutlet weak var mapView: GMSMapView!
  private var locationManager: CLLocationManager?
  private var currentLocation: CLLocation?
  private var placesClient: GMSPlacesClient!
  private var zoomLevel: Float = 15.0
  /// 初期描画の判断に利用
  private var initView: Bool = false

  override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view, typically from a nib.

    // GoogleMapの初期化
    self.mapView.isMyLocationEnabled = true
    self.mapView.mapType = GMSMapViewType.normal
    self.mapView.settings.compassButton = true
    self.mapView.settings.myLocationButton = true
    self.mapView.delegate = self

    // 位置情報関連の初期化
    self.locationManager = CLLocationManager()
    self.locationManager?.desiredAccuracy = kCLLocationAccuracyBest
    self.locationManager?.requestAlwaysAuthorization()
    self.locationManager?.distanceFilter = 50
    self.locationManager?.startUpdatingLocation()
    self.locationManager?.delegate = self

    self.placesClient = GMSPlacesClient.shared()
  }

  override func didReceiveMemoryWarning() {
    super.didReceiveMemoryWarning()
    // Dispose of any resources that can be recreated.
  }

  // MARK: CLLocationManagerDelegate
  func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
    if !self.initView {
      // 初期描画時のマップ中心位置の移動
      let camera = GMSCameraPosition.camera(withTarget: (locations.last?.coordinate)!, zoom: self.zoomLevel)
      self.mapView.camera = camera
      self.locationManager?.stopUpdatingLocation()
      self.initView = true
    }
  }
}
```

これは`viewDidLoad`時に`startUpdatingLocation()`で位置情報の取得を開始し、取得したタイミングである`didUpdateLocations`内で位置情報を`camera`に設定しています。  
(もっと良い方法がありそうな気もしますが、一旦これで様子見...)  

### まとめ
さて如何でしたでしょうか？  
これから`Google Maps SDK for iOS`をバシバシ使っていきたいと思っているので理解が進み次第、続きを書いていきたいと思います。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
