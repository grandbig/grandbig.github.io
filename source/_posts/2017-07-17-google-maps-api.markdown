---
layout: post
title: "Geocoding APIとDirections APIを使ってみよう！"
date: 2017-07-17 22:32
comments: true
categories: ios swift google api
---

### はじめに
本日はGoogleが提供している[Geocoding API](https://developers.google.com/maps/documentation/geocoding/intro?hl=ja)と[Directions API](https://developers.google.com/maps/documentation/directions/?hl=ja)をiOSで使ってみた話を書きます。  

まずは、Google Cloud Platform > API Managerから **Geocoding API** と **Directions API** を有効にしましょう。  

![API Manager](/images/google_api_1.png)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### Geocoding APIの利用
Googleから提供されたAPIを利用し、それがJSON形式が返却されるため、`Alamofire`と`SwiftyJSON`をあわせて利用します。  

因みに、APIキーは[Google Maps SDK for iOSを導入してみよう！](http://grandbig.github.io/blog/2017/06/18/google-maps-sdk/)で説明した通り`key.plist`に記載することでGitHubにアップすることを避けています。  

今回実装する処理は **住所を緯度/経度に変換する処理** になります。  

```objective-c
import CoreLocation
import Alamofire
import SwiftyJSON

class Geocoding {

  /// API Key
  private var apiKey: String = String()
  /// Geocoding APIのベースURL
  private let baseURL: String = "https://maps.googleapis.com/maps/api/geocode/json?language=ja"

  /// 初期化処理
  init() {
    if let path = Bundle.main.path(forResource: "key", ofType: "plist") {
      if let dic = NSDictionary(contentsOfFile: path) as? [String: Any] {
        if let apiKey = dic["googleWebApiKey"] as? String {
          self.apiKey = apiKey
        }
      }
    }
  }

  /**
   ジオコーディング

   - parameter address: 住所
   - parameter completion: 緯度/経度を返すcallback
   */
  func geocoding(address: String, completion: @escaping ((CLLocationCoordinate2D) -> Void)) {
    let requestURL = "\(baseURL)&key=\(String(describing: self.apiKey))"
    Alamofire.request(requestURL, method: .get, parameters: ["address": address], encoding: URLEncoding.default, headers: nil).responseJSON { response in
      let json = JSON(response.result.value as Any)

      guard let latitude = json["results"][0]["geometry"]["location"]["lat"].double else {
        return
      }

      guard let longitude = json["results"][0]["geometry"]["location"]["lng"].double else {
        return
      }

      completion(CLLocationCoordinate2D.init(latitude: latitude, longitude: longitude))
    }
  }  
}
```

### Directions APIの利用
ここで実装する処理は **開始地点から終了地点までの道順を取得する処理** になります。  

```objective-c
import CoreLocation
import Alamofire
import SwiftyJSON

class Direction {

  /// API Key
  private var apiKey: String = String()
  /// Geocoding APIのベースURL
  private let baseURL: String = "https://maps.googleapis.com/maps/api/directions/json?language=ja&mode=walking"

  /// 初期化処理
  init() {
    if let path = Bundle.main.path(forResource: "key", ofType: "plist") {
      if let dic = NSDictionary(contentsOfFile: path) as? [String: Any] {
        if let apiKey = dic["googleWebApiKey"] as? String {
          self.apiKey = apiKey
        }
      }
    }
  }

  /**
   目的地までの道順を取得

   - parameter from: 開始地点
   - parameter to: 終了地点
   - parameter completion: 道順を返すcallback
   */
  func getRoutes(from: CLLocationCoordinate2D, to: CLLocationCoordinate2D, completion: @escaping ((JSON) -> Void)) {
    let requestURL = "\(baseURL)&key=\(String(describing: self.apiKey))"
    let parameters = ["origin": "\(from.latitude),\(from.longitude)", "destination": "\(to.latitude),\(to.longitude)"]
    Alamofire.request(requestURL, method: .get, parameters: parameters, encoding: URLEncoding.default, headers: nil).responseJSON { response in
      let json = JSON(response.result.value as Any)
      let steps = json["routes"][0]["legs"][0]["steps"]

      completion(steps)
    }
  }
}
```

### まとめ
今回はGoogleが提供している **Geocoding API** と **Directions API** について見てみました。  
Google Mapを利用するサービスを考えると案外必要となる場面が多いんですよね。  
知っておいて損はないかと思います。  
本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
