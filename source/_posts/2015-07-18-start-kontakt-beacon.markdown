---
layout: post
title: "Kontakt.ioのSmart Beaconを使ってみた"
date: 2015-07-18 22:40
comments: true
categories: ios iBeacon
---

####Kontakt.ioのSmart Beaconを使ってみよう
さて、久しぶりの更新です。  
実は先日、以前から気になっていた[Kontakt.io Smart Beacon](https://store.kontakt.io/our-products/1-bluetooth-beacon.html)を購入してみました。  
あまり海外の製品を購入することがないので、届くか心配でしたが、ばっちり自宅に届きました笑  

なぜ、Kontakt.ioのSmart Beaconに着目していたかというと、筆者が知る限り、最も発信出力(TxPower)を弱くすることが可能なBeaconだからです。  
[仕様書](https://support.kontakt.io/hc/en-gb/articles/201621521-Transmission-power-Range-and-RSSI)を見てみると、出力を-30[dBm]にまで下げることができ、その電波距離は実に2[m]程度らしいです。  

また、SDKが充実しており、発信出力を自由に変更できるところもポイント高いですね。  
さて、今回は基本的な実装から見て行きましょう！(何回かに分けて記事にしようと思います。)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####SDKをプロジェクトに追加
まずは、SDKを使える状態に持っていきます。  

#####Objective-C編
CocoaPodsで提供されていますので、簡単です。  

1. Projectファイルと同じ階層に`Podfile`を作成します
2. `pod 'KontaktSDK'`と記載します
3. `pod install`を実行します

以上でプロジェクトへの追加完了です。  

#####Swift編
CocoaPods使いましょう！  

1. Projectファイルと同じ階層に`Podfile`を作成します
2. `pod 'KontaktSDK'`と記載します
3. `pod install`を実行します
4. Bridging-Headerファイルを作成します
`ProjectName-Bridging-Header.h`ファイルを作成し、Build Settings > Swift Compiler > Objective-C Bridging Headerにパスを設定
5. Bridging-Headerファイルに必要なライブラリを`import`します

下記は、Buid Settingsの設定です。  
![Bridging-Headerファイルの設定](/images/kontakt1.png)  

下記はBridging-Headerファイルの中身です。  

```objective-c
#ifndef KontaktSwiftTest_KontaktSwiftTest_Bridging_Header_h
#define KontaktSwiftTest_KontaktSwiftTest_Bridging_Header_h

#import <Foundation/Foundation.h>
#import <CoreLocation/CoreLocation.h>
#import <CoreBluetooth/CoreBluetooth.h>
#import "KontaktSDK.h"

#endif

```

####Kontakt.ioのBeaconを検知してみよう
これで準備が整ったので、実際にソースを書いて、Beaconを検知してみましょう！  

#####Objective-C編
ViewController.mにどんどん書いていきます。  

```objective-c
// ViewController.m
#import "ViewController.h"
#import "KontaktSDK.h"

@interface ViewController ()<KTKLocationManagerDelegate>

@property(strong, nonatomic) KTKLocationManager *lm;

@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];

	_lm = [KTKLocationManager new];
	_lm.delegate = self;

	if ([KTKLocationManager canMonitorBeacons]) {
		KTKRegion *region = [KTKRegion alloc] init];
		region.uuid = @"AF41A130-E105-4F13-9483-316B7101B0A9";
		[_lm setRegions:@[region]];]
	}
}

- (void)viewDidAppear:(BOOL)animated {
	[super viewDidAppear:animated];

	[_lm startMonitoringBeacons];
}

- (void)viewDidDisappear:(BOOL)animated {
	[super viewDidDisappear:animated];
	
	[_lm stopMonitoringBeacons];
}

- (void)didReceiveMemoryWarning {
	[super didReceiveMemoryWarning];
}

#pragma mark - KTKLocationManagerDelegate
- (void)locationManager:(KTKLocationManager *)locationManager didChangeState:(KTKLocationManagerState)state withError:(NSError *)error {
	if(state == KTKLocationManagerStateFailed) {
		NSLog(@"Something went wrong with your Location Services settings. Check OS settings.");
	}
}

- (void)locationManager:(KTKLocationManager *)locationManager didEnterRegion:(KTKRegion *)region {
	NSLog(@"Enter region %@", region.uuid);
}

- (void)locationManager:(KTKLocationManager *)locationManager didExitRegion:(KTKRegion *)region {
	NSLog(@"Exit region %@", region.uuid);
}

- (void)locationManager:(KTKLocationManager *)locationManager didRangeBeacons:(NSArray *)beacons {
	NSLog(@"Ranged beacons count: %lu", [beacons count]);
}

@end

```

#####Swift編
`ViwController.swift`にどんどん書いていきます。  

```objective-c
// ViewController.swift
import UIKit

class ViewController: UIViewController, KTKLocationManagerDelegate {

	let lm:KTKLocationManager = KTKLocationManager()
	
	override func viewDidLoad() {
		super.viewDidLoad()

		if(KTKLocationManager.canMonitorBeacons()) {
			let region:KTKRegion = KTKRegion()
			region.uuid = "AF41A130-E105-4F13-9483-316B7101B0A9"
			lm.setRegions([region])
			self.lm.delegate = self
		}
	}

	override func viewDidAppear(animated: Bool) {
		super.viewDidAppear(animated)
		
		lm.startMonitoringBeacons()
	}

	override func viewDidDisappear(animated: Bool) {
		super.viewDidDisappear(animated)
		
		lm.stopMonitoringBeacons()
	}

	override func didReceiveMemoryWarning() {
		super.didReceiveMemoryWarning()
	}

	func locationManager(locationManager: KTKLocationManager!, didChangeState state: KTKLocationManagerState, withError error: NSError!) {
		if(state == .Failed) {
			println("Something went wrong with your Location Services settings. Check OS settings.")
		}
	}

	func locationManager(locationManager: KTKLocationManager!, didEnterRegion region: KTKRegion!) {
		println("Enter region \(region.uuid)")
	}

	func locationManager(locationManager: KTKLocationManager!, didExitRegion region: KTKRegion!) {
		println("Exit region \(region.uuid)")
	}

	func locationManager(locationManager: KTKLocationManager!, didRangeBeacons beacons: [AnyObject]!) {
		println("Ranged beacons count: \(beacons.count)")
	}
}
```

これでKontakt.ioのSmart Beaconを検知することができるようになりました。  
次回はAPI連携して、Beaconのパラメータを変えてみたいと思います。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
