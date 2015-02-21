---
layout: post
title: "CLLocationManagerDelegateのdidUpdateToLocationがDeprecated!?"
date: 2014-04-05 11:25
comments: true
categories: ios location
---

###didUpdateToLocationではなくdidUpdateLocationsを使うべき!?
久々に更新します。今日は位置情報サービスで変更があった点に注目してみます。筆者は昨日まで、この変更を気にせずに普通にこれまで通りの方法で位置情報の取得をしていました。  
しかし、よくよく[AppleのCLLocationManagerDelegate Protocol Reference](https://developer.apple.com/library/ios/documentation/CoreLocation/Reference/CLLocationManagerDelegate_Protocol/CLLocationManagerDelegate/CLLocationManagerDelegate.html)を見てみると、『locationManager:didUpdateToLocation:fromLocation:』に**Deprecated in iOS 6.0**の文字が...。  
そして代わりに『locationManager:didUpdateLocations:』というものがありました。  
う〜む...どうやら今はこちらを使った方が良いようですね。  

<!--more-->

しかし、最新のXcode5.1でDeployment Targetを6.0に設定して下記をソースに書いたとしてもDeprecatedとWarningが出ることはありません。  
```objective-c
- (void)locationManager:(CLLocationManager*)manager
	didUpdateToLocation:(CLLocation *)newLocation
		   fromLocation:(CLLocation *)oldLocation
{
	CLLocationDegrees lat = [newLocation coordinate].latitude; // 緯度
	CLLocationDegrees lng = [newLocation coordinate].longitude; // 経度
	CLLocationDistance altitude = newLocation.altitude; // 高度
	CLLocationDirection course = newLocation.course; // 方角
	CLLocationSpeed speed = newLocation.speed; // 速度
	CLLocationAccuracy horizontalAccuracy = newLocation.horizontalAccuracy; //水平方向の精度
	CLLocationAccuracy verticalAccuracy = newLocation.verticalAccuracy; //垂直方向の精度
}
```

これではDeprecatedになっていることに気づかないですよね...。ま、新iOSがリリースされるごとにリファレンスをきちんと読むべきってことですね。  
はい！では今度からこう書きましょう。  

```objective-c
- (void)locationManager:(CLLocationManager *)manager
	 didUpdateLocations:(NSArray *)locations
{
	CLLocation *currentLocation = locations.lastObject;
	CLLocationDegrees lat = currentLocation.coordinate.latitude; // 緯度
	CLLocationDegrees lng = currentLocation.coordinate.longitude; // 経度
	CLLocationDistance altitude = currentLocation.altitude; // 高度
	CLLocationDirection course = currentLocation.course; // 方角
	CLLocationSpeed speed = currentLocation.speed; // 速度
	CLLocationAccuracy horizontalAccuracy = currentLocation.horizontalAccuracy; // 水平方向の精度
	CLLocationAccuracy verticalAccuracy = currentLocation.verticalAccuracy; // 水ty区方向の精度
}
```
古い方で位置情報を取得しても少なくともiOS6では問題ありませんでした。が、iOS7ではまだ未検証なので、試してみます。とは言え、新しい方を使うべきだと思いますが...。  
と言ったところで本日はここまで。

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
