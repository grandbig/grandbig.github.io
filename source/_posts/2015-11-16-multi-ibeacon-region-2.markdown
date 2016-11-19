---
layout: post
title: "iOSで検知可能なiBeaconの数について(別UUIDの場合)"
date: 2015-11-16 00:15
comments: true
categories: ios iBeacon
---

####iOSで検知可能なCLBeaconRegionの数について
さて、今日は久しぶりにiBeaconについて書きます。  
以前、[複数のiBeaconを検知しよう！(別UUIDの場合)](http://grandbig.github.io/blog/2015/06/14/multi-ibeacon-region/)で別UUIDの複数のiBeaconを検知する方法について書きました。  
今回は、その限界数について見ていきたいと思います。  

[位置情報とマッププログラミングガイド](https://developer.apple.com/jp/documentation/LocationAwarenessPG.pdf)を読むと、地理的領域の観測は最大20個まで可能とされています。  
しかし、iBeacon領域の観測について最大数が書かれていませんでした。  
Googleで調べてもなかなか出てこなかったので、自分で調べることにしました。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

結果から言うと、iBeacon領域の観測は **最大20個** まで可能でした。  
(地理的領域の観測と同じでした。)  

因みに、下記ソースで確認しています。  

```objective-c
import UIKit
import CoreLocation

class ViewController: UIViewController, CLLocationManagerDelegate {
	
	var lm:CLLocationManager?
	var uuids:[String]?

	override func viewDidLoad() {
		super.viewDidLoad()

		self.lm = CLLocationManager()
		self.lm?.delegate = self

		self.uuids = [ ここにUUIDの文字列を20個以上用意 ]

		self.lm?.requestAlwaysAuthorization()
	}

	override func didReceiveMemoryWarning() {
		super.didReceiveMemoryWarning()
	}

	func locationManager(manager: CLLocationManager, didChangeAuthorizationStatus status: CLAuthorizationStatus) {
		// 位置情報サービスの許可状態が変更した場合に呼び出される処理
		if(status == .AuthorizedAlways) {
			for(var i=0; i < self.uuids.count; i++) {
				let uuidString = self.uuids[i]
				let uuid = NSUUID.init(UUIDString: uuidString)
				let beaconRegion = CLBeaconRegion.init(proximityUUID: uuid!, identifier: ("beaconRegion"+String(i)))
				self.lm?.startMonitoringForRegion(beaconRegion)
			}
		}
	}

	func locationManager(manager: CLLocationManager, didStartMonitoringForRegion region: CLRegion) {
		// Regionのモニタリングが開始したときに呼び出される処理
		print("\(region)")
	}

	func locationManager(manager: CLLocationManager, didDetermineState state: CLRegionState, forRegion region: CLRegion) {
		// Regionに対する状態が変化したときに呼び出される処理
		switch(state) {
		case .Inside:
			print("Inside: \(region)")
			break
		case .Outside:
			print("Outside: \(region)")
			break
		case .Unknown:
			print("Unknown: \(region)")
			break
		}
	}
}
```

`viewDidLoad`内でuuidsの初期化を行っていますが、ここで20個UUIDを設定すると、`didDetermineState`内でログが20回出力されました。  
しかし、21個以上UUIDを設定した場合は、先ほどと同じく20回のログ出力となり、21個目以降に設定したUUIDに関してはログ出力されませんでした。  
つまり、先に設定したUUIDが有効で、21個目以降のUUIDは無効ということになります。  

複数のUUIDを大量に使う際には気をつけましょう。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
