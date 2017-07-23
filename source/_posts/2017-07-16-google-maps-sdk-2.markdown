---
layout: post
title: "Google Maps SDK for iOSを使ってみよう！"
date: 2017-07-16 23:43
comments: true
categories: ios swift map google
---

### はじめに
前回、[Google Maps SDK for iOSを導入してみよう！](http://grandbig.github.io/blog/2017/06/18/google-maps-sdk/)について説明しましたが、今回はもう一歩踏み込んで使い方を見ていこうと思います。  

これまた本家の[Google スタートガイド](https://developers.google.com/maps/documentation/ios-sdk/start?hl=ja)を見ればできることも多いのですが、見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### Google Mapにマーカを配置する
Google Mapを使う上で必ずと言っていいほど利用する機能である「マーカの配置」を見ていきましょう。  
これは実に簡単で「マーカを配置するメソッド」はたった下記だけで実装できます。  

```objective-c
/**
  マップにマーカを設置する処理

  - parameter title: マーカのタイトル
  - parameter coordinate: 位置
  - parameter iconName: アイコン名
  - parameter completion: Callback
*/
private func putMarker(title: String?, coordinate: CLLocationCoordinate2D, iconName: String?, completion: @escaping ((GMSMarker) -> Void)) {
  // マーカの生成
  let marker = GMSMarker()
  marker.title = title
  marker.position = coordinate
  if iconName != nil {
    // アイコン名が指定されている場合は画像を設定
    marker.icon = UIImage.init(named: iconName!)
  }
  marker.map = self.mapView
  completion(marker)
}
```

### Google Mapからマーカを削除する
逆にGoogle Mapからマーカを削除する場合はどうするかを見ていきます。  
これも簡単なので、下記のように実装できます。  

```objective-c
/**
  マップからマーカを削除する処理

  - parameter marker: マーカ
*/
private func deleteMarker(marker: GMSMarker) {
  marker.map = nil
}
```

### 2つのマーカが入る縮尺にGoogle Mapを変更する
こちらはGoogle Mapを捉えるカメラの位置を移動することで実現可能です。  

```objective-c
@IBOutlet weak var mapView: GMSMapView!

<省略>

/**
  現在地と指定した場所の両方が入るようにマップの縮尺を変更する処理

  - parameter coordinate: 場所
*/
private func changeCameraPosition(fromCoordinate: CLLocationCoordinate2D, toCoordinate: CLLocationCoordinate2D) {
  let bounds = GMSCoordinateBounds(coordinate: fromCoordinate, coordinate: toCoordinate)
  let margin: CGFloat = 50.0  // 上下左右に設定するマージン
  guard let camera = self.mapView.camera(for: bounds, insets: UIEdgeInsets(top: margin, left: margin, bottom: margin, right: margin)) else {
    return
  }
  self.mapView.camera = camera
}
```

### Google Mapに線を描画する
これもそんなに難しくありません。  

```objective-c
@IBOutlet weak var mapView: GMSMapView!
private var routePath: GMSPolyline = GMSPolyline()

<省略>

/**
  マップへの線描画

  - parameter fromCoordinate: 起点位置
  - parameter toCoordinate: 終点位置
*/
private func drawPolyline(fromCoordinate: CLLocationCoordinate2D, toCoordinate: CLLocationCoordinate2D) {

  let path = GMSMutablePath()
  path.add(fromCoordinate)
  path.add(toCoordinate)

  self.routePath = GMSPolyline(path: path)
  self.routePath.strokeWidth = 3.0
  self.routePath.map = self.mapView
}
```

### Google Mapから線を削除する
先程描画した線を削除するには下記で実行できます。  

```objective-c
private var routePath: GMSPolyline = GMSPolyline()

<省略>

/**
  マップへの描画線を削除する処理
*/
private func clearRoutePath() {
  self.routePath.map = nil
}
```

### 緯度/経度とピクセル座標の相互変換
これは利用ケースが限られるかもしれませんが、覚えておくと役に立つ処理です。  
Google Mapは複数のマップ座標を扱うことができます。  

- 緯度/経度を用いて地球上にプロット  
- 世界座標: メルカトル図法を用いて緯度/経度を地図に変換した座標  
- ピクセル座標: 世界座標を指定したズームレベルで変換した座標  
- タイル座標: 地図を複数の画像に分けたときの座標  

ほとんどの場合は緯度/経度をマップにプロットすると思いますが、  
筆者は先日ピクセル座標を利用する場面がありました。  

それは、「Google Mapの上に透過Viewが載せられた状態でGoogle Mapにマーカを配置する」というものでした。
今回はこれを例に変換方法を見ていきましょう。  

#### ピクセル座標を緯度/経度に変換
これはGoogle Maps SDKに用意されています。  

```objective-c
let pressPoint = CGPoint(x: 100, y: 200)
let coordinate = self.mapView.projection.coordinate(for: pressPoint)
```

たったのこれだけでピクセル座標を緯度/経度に変換できるんです。  

#### 緯度/経度をピクセル座標に変換
これもGoogle Maps SDKに用意されています。  

```objective-c
let coordinate = CLLocationCoordinate2D(latitude: 35, longitude: 139)
let point = self.mapView.projection.point(for: coordinate)
```

### まとめ
さて如何でしたでしょうか？  
JavaScriptでGoogle Maps APIを利用していた方々も臆することなく使えるような簡単さだと思います。  
次回はGeocoding APIやDirection APIを見ていきたいと思います。  

と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
