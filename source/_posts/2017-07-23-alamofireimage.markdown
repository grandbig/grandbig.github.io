---
layout: post
title: "AlamofireImageを使ってみよう！"
date: 2017-07-23 15:35
comments: true
categories: ios swift
---

### はじめに
本日は何気に今まで使ってこなかった[AlamofireImage](https://github.com/Alamofire/AlamofireImage)についてメモ書きです。  
キャッシュコントロールやら同期/非同期での画像取得など考えなくても良いというのはパワー的にかなり楽になりますね。  
ということで早速見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### API経由で取得した画像URLを使ってUIImageViewに画像を表示する
#### テストとして利用するAPI
今回はテスト用APIとして[ホットペッパーのグルメサーチAPI](https://webservice.recruit.co.jp/hotpepper/reference.html)を利用しました。  

ホットペッパーAPIを利用するには新規登録して`API Key`をゲットする必要があります。  
また、前提として取得した`API Key`は`key.plist`に書いているとします。  

![API Keyをkey.plistに書き出し](/images/alamofireimage_1.png)  

上記準備をした上で下記クラスを作成しました。  

```objective-c
// HotpepperAPI.swift
import Foundation
import CoreLocation
import Alamofire
import SwiftyJSON

/**
 ホットペッパーAPI
 */
class HotpepperAPI {
  /// API Key
  private var apiKey: String = String()
  /// ホットペッパーAPIのベースURL
  private let baseURL: String = "https://webservice.recruit.co.jp/hotpepper/gourmet/v1/"

  /// 初期化処理
  init() {
    if let path = Bundle.main.path(forResource: "key", ofType: "plist") {
      if let dic = NSDictionary(contentsOfFile: path) as? [String: Any] {
        if let apiKey = dic["hotpepperApiKey"] as? String {
          self.apiKey = apiKey
        }
      }
    }
  }

  /**
   ホットペッパーグルメサーチAPI

   - parameter coordinate: 位置
   - parameter completion: レストラン情報を返却するcallback
   */
  func searchRestaurant(coordinate: CLLocationCoordinate2D, completion: @escaping ((JSON) -> Void)) {
    let parameters = ["key": self.apiKey, "format": "json", "lat": coordinate.latitude, "lng": coordinate.longitude, "range": 2] as [String : Any]
    Alamofire.request(baseURL, method: .get, parameters: parameters, encoding: URLEncoding.default, headers: nil).responseJSON { response in
      let json = JSON(response.result.value as Any)
      let result = json["results"]["shop"]

      completion(result)
    }
  }
}
```

#### テストとして用意するUIImageView
今回はテストとして **マップにプロットしたマーカをタップしたときに表示するInfoWindow内にUIImageViewを用意する** ようにしました。  

【準備事項】  
- Google Maps SDK for iOSをマップとして利用  
- マーカタップ時に表示されるInfoWindowをカスタム化  

表示するViewとしては下図のようになります。  
![MarkerInfoContentsView.xib](/images/alamofireimage_2.png)  

#### AlamofireImageの用意
では肝心な`AlamofireImage`の導入を見てみましょう。  
`CocoaPods`で簡単に導入が可能です。  

```objective-c
# Podfile
use_frameworks!
platform :ios, '10.0'

target "SampleApp" do
  # Normal libraries
  ...
  pod 'AlamofireImage', '~> 3.1'
end

post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['SWIFT_VERSION'] = '3.0'
    end
  end
end
```

#### マーカタップ時に表示するInfoWindowに画像を表示する
まずは`InfoWindow`をカスタム化したクラスである`MarkerInfoContentsView.swift`のソースコードを書きます。  

```objective-c
// MarkerInfoContentsView.swift
import Foundation
import UIKit
import AlamofireImage

class MarkerInfoContentsView: UIView {

  @IBOutlet weak var shopName: UILabel!
  @IBOutlet weak var categoryName: UILabel!
  @IBOutlet weak var shopImage: UIImageView!

  override init(frame: CGRect) {
    super.init(frame: frame)
    self.xibViewSet()
  }

  required init?(coder aDecoder: NSCoder) {
    super.init(coder: aDecoder)!
    self.xibViewSet()
  }

  internal func xibViewSet() {
    if let view = Bundle.main.loadNibNamed("MarkerInfoContentsView", owner: self, options: nil)?.first as? UIView {
      view.frame = self.bounds
      self.addSubview(view)
    }
  }

  /**
    データの設定処理

    - parameter shopName: 店舗名
    - parameter categoryName: カテゴリ名
    - parameter shopImageURLString: 画像URL
   */
  func setData(shopName: String?, categoryName: String?, shopImageURLString: String?) {
    // 店舗名の設定
    if let shopNameTextCount = shopName?.characters.count, shopNameTextCount > 0 {
      self.shopName.text = shopName
    } else {
      self.shopName.text = "店舗名不明"
      self.shopName.textColor = UIColor.gray
    }
    // 詳細説明の設定
    if let categoryNameTextCount = categoryName?.characters.count, categoryNameTextCount > 0 {
      self.categoryName.text = categoryName
    } else {
      self.categoryName.text = "カテゴリ不明"
      self.categoryName.textColor = UIColor.gray
    }
    // 画像の設定
    if let shopImageURLStringTextCount = shopImageURLString?.characters.count, shopImageURLStringTextCount > 0 {
      if let shopImageURL = URL(string: shopImageURLString!) {
        self.shopImage.af_setImage(withURL: shopImageURL, placeholderImage: UIImage(named: "NoImageIcon"))
      } else {
        self.shopImage.image = UIImage(named: "NoImageIcon")
      }
    } else {
      self.shopImage.image = UIImage(named: "NoImageIcon")
    }    
  }
}
```

実際に`AlamofireImage`を利用して画像URLから取得した画像データを格納している箇所は、  

```objective-c
self.shopImage.af_setImage(withURL: shopImageURL, placeholderImage: UIImage(named: "NoImageIcon"))
```

になります。  
続いて、`ViewController.swift`での`GMSMapViewDelegate`部分の処理を抜粋して書きます。  

```objective-c
// ViewController.swift
extension ViewController: GMSMapViewDelegate {
  func mapView(_ mapView: GMSMapView, markerInfoWindow marker: GMSMarker) -> UIView? {
    guard let cMarker = marker as? CustomGMSMarker else {
      return nil
    }
    cMarker.tracksInfoWindowChanges = true
    let view = MarkerInfoContentsView(frame: CGRect(x: 0, y: 0, width: 250, height: 265))
    view.setData(shopName: cMarker.shopName, categoryName: cMarker.categoryName, shopImageURLString: cMarker.imageURL)
    return view
    }
}
```

重要なのは、 **`cMarker.tracksInfoWindowChanges = true`** です。  
これを書かないと **画像URLから画像データを取得したタイミングで`InfoWindow`の画像を更新する**ということができなくなります。  
(`placeholderImage`として用意した画像がずっと表示されてしまいます。)  

筆者はここでドハマリして試行錯誤してしまいました。  
非同期で画像データを取得しに行っているので、データ取得前に`InfoWindow`の描画処理に進んでしまうということはわかるのですが、どうすれば想定した挙動が実現できるのか悩みました。  
ですが、蓋を開けてみれば何ということもなかったんですよね。  

### まとめ
さて、如何でしたでしょうか？  
画像取得/キャッシュ関連のOSSライブラリは多種多様なものが出ており、好き嫌いもあるかもしれませんが、筆者は通信ライブラリに`Alamofire`を使うことが多いため、`AlamofireImage`も嫌いではないんですよね。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
