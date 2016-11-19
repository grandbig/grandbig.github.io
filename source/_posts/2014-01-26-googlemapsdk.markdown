---
layout: post
title: "iOSでMapにGIFアニメーションの画像を表示したい"
date: 2014-01-26 23:29
comments: true
categories: ios map
---

###MAPにGIFアニメーション画像を表示したい
最近、ネイティブアプリ開発を再び始めました。そこでマップ上にアニメーションGIF画像を表示しようと思ったのですが、これがなかなかうまくいかないんですね。  
そのときのことについて今日は書きます。  

iOSのMapライブラリにはApple標準提供の**MapKit**とGoogleが提供する**Google Maps SDK for iOS**があります。  
当然のことながら標準のMapKitの方が導入は簡単です。(Google Mapsの利用にはそれなりに導入手順があります。ただ、ここで挫折しなければ実はかなり使いやすいのではないかと。)

Google Mapsの導入手順については下記を参考にすると良いと思います。  
[Google Maps SDK for iOSの導入手順 - Qiita](http://qiita.com/shu223/items/bfb5ef3e45682c2bb763)  
[Google Maps SDK for iOSの使い方 - 目くじら日記](http://dev.eyewhale.com/archives/157)  

<!--more-->

####Apple標準提供のMapKitを使ってみる
さて、まずは当然のことながらMapKitで試してみることにしました。そもそもMapKitライブラリを使用すると簡単にはピンの画像を変えられません。  
詳しくは[Mapに好きな画像を配置しよう！](http://grandbig.github.io/blog/2013/09/28/put-annotation/)を見てください。  

MapKitにGIFアニメーション画像を配置するべく、LBGIFImageを利用しました。非常に簡単に利用できるので、ぜひ[LBGIFImage - GitHub](https://github.com/larcus94/LBGIFImage)からダウンロードしてみてください。
しかし、MapKitではGIFアニメーション画像を表示することができませんでした。アノテーション(Mapにさすピン)をGIFアニメーションで置き換えることができないのか定かではないのですが、うまくいきませんでした。

####Google Maps SDK for iOSを使ってみる
次にもしや**Google Maps SDK for iOS**ならできるのでは！？と思いつき試してみました。  
結果から言いますと、超簡単にできました！！  

```
// マーカーの作成
GMSMarker *marker = [[GMSMarker alloc] init];
// 位置の設定
marker.position = CLLocationCoordinate2DMake(35.659352, 139.776228);
// マーカーを地図にプロット
marker.map = mapView;
// GIFアニメーションをUIImageに設定
UIImage* image = [UIImage animatedGIFNamed:@"GIF画像名(.gifは含まない)"];
// マーカーにその画像を設定
marker.icon = image;
```
これだけでできます。(もちろんLBGIFImageライブラリはインポートしてください)

標準提供のMapKitを利用しているアプリでGIFアニメーションをマップ上に表示しているアプリもあるので、やり方はあるのと思います。
ただ、Google Maps SDKはとても使いやすそうなので、こっち使おうかな。

