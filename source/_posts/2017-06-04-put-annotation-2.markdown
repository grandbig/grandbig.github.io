---
layout: post
title: "Mapに好きな画像を配置しよう！ Swift編"
date: 2017-06-04 21:24
comments: true
categories: ios swift map
---

### はじめに
実に3年半ほど前のブログを始めた当初、[Mapに好きな画像を配置しよう！](https://grandbig.github.io/blog/2013/09/28/put-annotation/)といった記事を書いたことがありました。  
駆け出しのiOSエンジニアであった当時の筆者はお世辞にもObjective-CやiOS自体について詳しいとは言い難きスキルレベルでした。  
(当時のブログ記事に不必要な記述があるかとは思いますが、あえてそのまま残しています。)  

それから月日を経て、Swiftで同じ実装をするタイミングがあったことで、本記事を書こうと思い、今に至ります。  
簡単な内容ではありますが、感慨深く書かせて頂いています笑  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### CustomAnnotationを作成しよう
当時と同じ手法で実装してみます。  

```objective-c
import Foundation
import MapKit

class CustomAnnotation:NSObject, MKAnnotation {
  public var coordinate: CLLocationCoordinate2D
  public var title: String?
  public var subtitle: String?

  init(coordinate: CLLocationCoordinate2D, title: String, subtitle: String) {
    self.coordinate = coordinate
    self.title = title
    self.subtitle = subtitle

    super.init()
  }
}
```

上記では、`MKAnnotation`を拡張し、`coordinate` / `title` / `subtitle`を初期化時に一斉に設定できるようなイニシャライザを用意しました。  

### Mapに画像を配置しよう
さて、ではMapに画像を配置する方法を見ていきます。  

```objective-c
import UIKit
import MapKit

class ViewController: UIViewController, MKMapViewDelegate {
  @IBOutlet weak var mapView: MKMapView!

  override func viewDidLoad() {
    super.viewDidLoad()

    // マップ関連の初期化処理
    self.mapView.delegate = self
    self.mapView.setUserTrackingMode(MKUserTrackingMode.followWithHeading, animated: true)

    // CustomAnnotationの初期化
    let ann = CustomAnnotation.init(coordinate: CLLocationCoordinate2D.init(latitude: 35.685623, longitude: 139.763153), title: "TEST", subtitle: "test")
    // CustomAnnotationをマップに配置
    self.mapView.addAnnotation(ann)
  }

  <省略>

  // MARK: MKMapViewDelegate
  func mapView(_ mapView: MKMapView, viewFor annotation: MKAnnotation) -> MKAnnotationView? {
    if (annotation is MKUserLocation) {
      // ユーザの現在地の青丸マークは置き換えない
      return nil
    } else {
      // CustomAnnotationの場合に画像を配置
      let identifier = "Pin"
      var annotationView: MKAnnotationView? = mapView.dequeueReusableAnnotationView(withIdentifier: identifier)
      if annotationView == nil {
        annotationView = MKAnnotationView.init(annotation: annotation, reuseIdentifier: identifier)
      }
      annotationView?.image = UIImage.init(named: "xxxx") // 任意の画像名
      annotationView?.annotation = annotation
      annotationView.canShowCallout = true  // タップで吹き出しを表示
      return annotationView
    }
  }
}
```

結果は次のようになります。  
![Mapに画像を表示](/images/annotationpractice6.png)  

### まとめ
昔は様々なサイトを参考にしながら、解読しながら書いていたソースがすんなりと書くことができました。  
今回の記事を通して、もっと他にもSwiftに書き直しても良い記事がありそうだなと思いました。
まあ、タイミング見てですかね...といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
