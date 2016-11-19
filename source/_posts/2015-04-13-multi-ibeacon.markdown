---
layout: post
title: "複数のiBeacon信号を利用してみよう！"
date: 2015-04-13 00:28
comments: true
categories: ios iBeacon
---

####複数のiBeaconを検知する方法
さて、本日はさらっと複数のiBeacon信号の検知について考えてみたいと思います。  
(これまでできるだろうという考えではあったものの、実際に試してみたことがなかったのでメモ書きです。)  
Central側ではiBeacon信号をキャッチしたときに、そのオブジェクトを配列の形で取得することができます。  
具体的には、  

```objective-c
// Beacon信号を検出した場合
- (void)locationManager:(CLLocationManager *)manager didRangeBeacons:(NSArray *)beacons inRegion:(CLBeaconRegion *)region
{
	if (beacons.count > 0) {
		NSLog(@"%@", beacons);
	}
}
```

のように **beacons** が **NSArray** の型で返ってきていますよね。  
単純にこれを使ってやればよいのです。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

実際にログを見てみると、  

```objective-c
2015-04-13 00:27:23.974 iBeaconSampleCentral[312:15030] (
	"CLBeacon (uuid:<__NSConcreteUUID 0x17403a6e0> 8D4DB809-032F-4771-96F3-99BD5C25F924, major:1, minor:2, proximity:1 +/- 0.07m, rssi:-35)",
	"CLBeacon (uuid:<__NSConcreteUUID 0x174037520> 8D4DB809-032F-4771-96F3-99BD5C25F924, major:1, minor:2, proximity:1 +/- 0.20m, rssi:-45)"
)
```

となっています。  
最寄りのiBeaconのみを使う方法ではできなかった何か新しいサービスを実現できるかもしれませんね。  
今日はここまで。  


<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
