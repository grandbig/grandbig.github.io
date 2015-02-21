---
layout: post
title: "Swiftを使って、iBeaconのCentralアプリを作ろう！！"
date: 2014-09-29 00:53
comments: true
categories: ios iBeacon Swift
---

###Swiftを使ってみよう
さて、本日はSwiftの勉強を兼ねて、iBeaconのCentralアプリを開発してみようと思います。もちろんiOS8に対応したアプリを開発します。  
これまで、筆者はまともにSwiftを触ってきませんでした。しかし、今後の動向を考えるに、Objective-Cだけでなく、Swiftでの開発もできるようになっておかないと技術者として危険な気がするのです。もし、数年後にXcode上でObjective-Cのサポートを終了するということになったら、途端にiPhoneアプリが開発できない技術者となってしまいます。  

とは言え、何もなしに書き方を勉強しようとしても辛いものがあります。なので、これまでObjective-Cで開発してきたアプリをSwiftに移行するという方法でSwiftを学んでいくこととしました。  
では、早速、開発を進めていきましょう！  

本ブログでは、Objective-CとSwiftでの書き方を比較しながら進めていきます。  
Objective-Cとの比較が不要な方は**Swiftの場合**の項目のみを読んでください。  

<!--more-->

####ライブラリのimport

* Objective-Cの場合  

```objective-c
// ViewController.hファイル
#import <UIKit/UIKit.h>
```

```objective-c
// ViewController.mファイル
#import "ViewController.h"
#import <CoreLocation/CoreLocation.h>
```

* Swiftの場合  

```
// ViewController.swiftファイル
import UIKit
import CoreLocation
```

###インスタンス変数の宣言

* Objective-Cの場合
Objective-Cの場合は`import`のすぐ下のinterface内で書きます。  

```objective-c
// ViewController.mファイル
@interface ViewController ()

@property (strong, nonatomic) IBOutlet CLLocationManager *locationManager;
@property (strong, nonatomic) IBOutlet NSUUID *proximityUUID;
@property (strong, nonatomic) IBOutlet CLBeaconRegion *beaconRegion;
@property (strong, nonatomic) IBOutlet CLBeacon *nearestBeacon;
@property (strong, nonatomic) IBOutlet NSString *str;

@end
```

* Swiftの場合  
Swiftの場合は`class ViewController`に全体が囲まれる書式になるため、`class ViewController`のすぐ下に書きます。  

```
class ViewController: UIViewController {

	@IBOutlet var locationManager:CLLocationManager?
	@IBOutlet var proximityUUID:NSUUID?
	@IBOutlet var beaconRegion:CLBeaconRegion?
	@IBOutlet var nearestBeacon:CLBeacon?
	@IBOutlet var str:NSString?

	.....

}
```

###Delegate Protocolの宣言

* Objective-Cの場合  

```objective-c
@interface ViewController() <CLLocationManagerDelegate, UIAlertViewDelegate> {
	.....
}
```

* Swiftの場合  

```
class ViewController: UIViewController, CLLocationManagerDelegate, UIAlertViewDelegate {
	.....
}
```

###viewDidLoadメソッド

* Objective-Cの場合  

```
- (void)viewDidLoad {
	[super viewDidLoad];

	if ([CLLocationManager isMonitoringAvailableForClass:[CLCircularRegion class]]) {
		self.locationManager = [[CLLocationManager alloc] init];
		self.locationManager.delegate = self;

		self.proximityUUID = [[NSUUID alloc] initWithUUIDString:@"UUID文字列を指定"];
		self.beaconRegion = [[CLBeaconRegion alloc] initWithProximityUUID: self.proximityUUID identifier:@"Objective-C iBeacon"];

		if ([self.locationManager respondsToSelector:@selector(requestAlwaysAuthorization)]) {
			[self.locationManager requestAlwaysAuthorization];
		} else {
			[self.locationManager startMonitoringForRegion: self.beaconRegion];
		}
	} else {
		UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"確認" message:@"お使いの端末ではiBeaconを利用できません。" delegate:self cancelButtonTitle:nil otherButtonTitles:@"OK", nil];
		[alert show];

	}
}
```

* Swiftの場合  

```
override func viewDidLoad() {
	super.viewDidLoad()

	if(CLLocationManager.isMonitoringAvailableForClass(CLCircularRegion)) {
		self.locationManager = CLLocationManager()
		self.locationManager?.delegate = self

		self.proximityUUID = NSUUID(UUIDString: "UUID文字列を指定")
		self.beaconRegion = CLBeaconRegion(proximityUUID: self.proximityUUID, identifier: "Swift iBeacon")

		if((self.locationManager?.respondsToSelector("requestAlwaysAuthorization")) != nil) {
			self.locationManager?.requestAlwaysAuthorization()
		} else {
			self.locationManager?.startMonitoringForRegion(self.beaconRegion)
		}
	} else {
		var alert:UIAlertView? = UIAlertView(title: "確認",message: "お使いの端末ではiBeaconを利用できません。", delegate: self, cancelButtonTitle: nil, otherButtonTitles: "OK")
		alert?.show()
	}
}
```

###自作メソッドの定義
ここではローカルプッシュを送信するメソッドを定義します。  

* Objective-Cの場合  

```
- (void)sendLocalNotificationForMessage:(NSString *)message
{
	UILocalNotification *localNotification = [UILocalNotification new];
	localNotification.alertBody = message;
	localNotification.fireDate = [NSDate date];
	localNotification.soundName = UILocalNotificationDefaultSoundName;
	[[UIApplication sharedApplication] scheduleLocalNotification:localNotification];
}
```

* Swiftの場合  

```
func sendLocalNotificationForMessage(message: NSString!) {
	var localNotification:UILocalNotification = UILocalNotification()
	localNotification.alertBody = message
	localNotification.fireDate = NSDate.date()
	localNotification.soundName = UILocalNotificationDefaultSoundName
	UIApplication.sharedApplication().scheduleLocalNotification(localNotification)
}
```

###CLLocationManagerのDelegateメソッド

* Objective-Cの場合  

```
// 領域計測が開始した場合
- (void)locationManager:(CLLocationManager *)manager didStartMonitoringForRegion:(CLRegion *)region
{
	[self sendLocalNotificationForMessage:@"Start Monitoring Region"];
}

// 領域に侵入した場合
- (void)locationManager:(CLLocationManager *)manager didEnterRegion:(CLRegion *)region
{
	[self sendLocalNotificationForMessage:@"Enter Region"];
		    
	if ([region isMemberOfClass:[CLBeaconRegion class]] && [CLLocationManager isRangingAvailable]) {
		[self.locationManager startRangingBeaconsInRegion:(CLBeaconRegion *)region];
	}
}

// 領域から退出した場合
- (void)locationManager:(CLLocationManager *)manager didExitRegion:(CLRegion *)region
{
	[self sendLocalNotificationForMessage:@"Exit Region"];
		    
	if ([region isMemberOfClass:[CLBeaconRegion class]] && [CLLocationManager isRangingAvailable]) {
		[self.locationManager stopRangingBeaconsInRegion:(CLBeaconRegion *)region];
	}
}

// 領域内にいるかいないかの判断
-(void)locationManager:(CLLocationManager *)manager didDetermineState:(CLRegionState)state forRegion:(CLRegion *)region{
	switch (state) {
		case CLRegionStateInside:
			if([region isMemberOfClass:[CLBeaconRegion class]] && [CLLocationManager isRangingAvailable]){
				NSLog(@"Enter %@",region.identifier);
				//Beacon の範囲内に入った時に行う処理を記述する
				[self sendLocalNotificationForMessage:@"Already Entering"];
			}
			break;

		case CLRegionStateOutside:
		case CLRegionStateUnknown:
		default:
			break;
	}
}

// 距離計測
- (void)locationManager:(CLLocationManager *)manager didRangeBeacons:(NSArray *)beacons inRegion:(CLBeaconRegion *)region
{
	if (beacons.count > 0) {
		//CLBeacon *nearestBeacon = beacons.firstObject;
		self.nearestBeacon = beacons.firstObject;

		NSString *rangeMessage;

		switch (self.nearestBeacon.proximity) {
			case CLProximityImmediate:
				rangeMessage = @"Range Immediate";
				break;
			case CLProximityNear:
				rangeMessage = @"Range Near";
				break;
			case CLProximityFar:
				rangeMessage = @"Range Far";
				break;
			default:
				rangeMessage = @"Range Unknown";
				break;
		}

		self.str = [[NSString alloc] initWithFormat:@"%f [m]", self.nearestBeacon.accuracy];
		[self sendLocalNotificationForMessage:self.str];
	}
}

// 領域観測に失敗した場合
- (void)locationManager:(CLLocationManager *)manager monitoringDidFailForRegion:(CLRegion *)region withError:(NSError *)error
{
	[self sendLocalNotificationForMessage:@"Exit Region"];
}

// 位置情報の許可設定
- (void)locationManager:(CLLocationManager *)manager didChangeAuthorizationStatus:(CLAuthorizationStatus)status
{
	if (status == kCLAuthorizationStatusNotDetermined) {
	} else if(status == kCLAuthorizationStatusAuthorizedAlways) {
		[self.locationManager startMonitoringForRegion: self.beaconRegion];
	} else if(status == kCLAuthorizationStatusAuthorizedWhenInUse) {
		[self.locationManager startRangingBeaconsInRegion:self.beaconRegion];
	}
}
```

* Swiftの場合  

```
// 領域観測が開始した場合
func locationManager(manager: CLLocationManager!, didStartMonitoringForRegion region: CLRegion!) {
	self.sendLocalNotificationForMessage("Start Monitoring Region")
}

// 領域に侵入した場合
func locationManager(manager: CLLocationManager!, didEnterRegion region: CLRegion!) {
	self.sendLocalNotificationForMessage("Enter Region")

	if(region.isMemberOfClass(CLBeaconRegion) && CLLocationManager.isRangingAvailable()) {
		self.locationManager?.startRangingBeaconsInRegion(region as CLBeaconRegion)
	}
}

// 領域から退出した場合
func locationManager(manager: CLLocationManager!, didExitRegion region: CLRegion!) {
	self.sendLocalNotificationForMessage("Enter Region")

	if(region.isMemberOfClass(CLBeaconRegion) && CLLocationManager.isRangingAvailable()) {
		self.locationManager?.stopRangingBeaconsInRegion(region as CLBeaconRegion)
	}
}

// 領域内にいるかどうかの判断
func locationManager(manager: CLLocationManager!, didDetermineState state: CLRegionState, forRegion region: CLRegion!) {
	switch(state) {
	case .Inside:
		if(region.isMemberOfClass(CLBeaconRegion) && CLLocationManager.isRangingAvailable()) {
			self.locationManager?.startRangingBeaconsInRegion(region as CLBeaconRegion)
		}
		break
	case .Outside:
		break
	case .Unknown:
		break
	default:
		break
	}
}

// 距離計測
func locationManager(manager: CLLocationManager!, didRangeBeacons beacons: [AnyObject]!, inRegion region: CLBeaconRegion!) {
	if(beacons.count > 0) {
		self.nearestBeacon = beacons[0] as? CLBeacon

		var proximity:CLProximity! = self.nearestBeacon?.proximity
		var bAccuracy:CLLocationAccuracy! = self.nearestBeacon?.accuracy
		var rangeMessage:NSString

		if(proximity == CLProximity.Immediate) {
			rangeMessage = "Range Immediate"
		} else if(proximity == CLProximity.Near) {
			rangeMessage = "Range Near"
		} else if(proximity == CLProximity.Far) {
			rangeMessage = "Range Far"
		} else if(proximity == CLProximity.Unknown) {
			rangeMessage = "Range Unknown"
		}

		self.str = "\(bAccuracy) [m]"
		self.sendLocalNotificationForMessage(self.str)
	}
}

// 領域観測に失敗した場合
func locationManager(manager: CLLocationManager!, monitoringDidFailForRegion region: CLRegion!, withError error: NSError!) {
	self.sendLocalNotificationForMessage("Exit Region")
}

// 位置情報の許可設定
func locationManager(manager: CLLocationManager!, didChangeAuthorizationStatus status: CLAuthorizationStatus) {
	if(status == .NotDetermined) {
	} else if(status == .Authorized) {
		self.locationManager?.startMonitoringForRegion(self.beaconRegion)
	} else if(status == .AuthorizedWhenInUse) {
		self.locationManager?.startRangingBeaconsInRegion(self.beaconRegion)
	}
}
```

以上でコーディング終了です。Objective-Cのときと同様にSwiftでもBeaconを検知することができるようになりました。  
Swiftの勉強がてら、こういった取り組みを進めていこうと思います。  
本日はここまで。  

参考:  
[SwiftとiBeaconを使ってお母さんが自分の部屋に近づいて来た事を警告するアプリをつくる](http://qiita.com/oggata/items/5de43d71692d1abcff7c)  
[Objective-C で書いたアプリを Swift で書き換える5ステップ](http://qiita.com/shu223/items/437d9a9a3502800a891b)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
