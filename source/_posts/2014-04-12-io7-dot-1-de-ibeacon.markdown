---
layout: post
title: "噂のiOS7.1でiBeaconを試してみよう！！"
date: 2014-04-12 23:41
comments: true
categories: ios iBeacon
---

###iOS7.1ならアプリのBackground起動も必要なし！？
さて、本日は最近いろいろなところで取り上げられているiOS7.1でのiBeaconについて実際に試してみました。  
iOS7から導入されたiBeaconですが、これまではアプリをBackgroundで起動していなければBeaconを検知することができませんでした。しかし、iOS7.1からアプリをBackgroundで起動していなくてもBeaconを検知できるようになったとビッグニュースになりました。でも、本当にそうなんでしょうか？と疑問に思った筆者はSampleアプリで試してみました。  

<!--more-->

下記がソースです。  
まずはBeacon機器としてiPadを用いるためにPeripheralソースから書きます。  
とりあえずProject作成の手順は  
1: BeaconPeripheralという名称で新規Project作成  
2: UIViewControllerを追加(ViewControllerという名称で作成しました)  
3: CoreBluetoothとCoreLocationライブラリを追加  
として下さい。  

ViewController.hのソース
```objective-c
#import <UIKit/UIKit.h>
#import <CoreBluetooth/CoreBluetooth.h>
#import <CoreLocation/CoreLocation.h>

@interface ViewController : UIViewController<CBPeripheralManagerDelegate>

@end
```
ViewController.mのソース  
```objective-c
#import "ViewController.h"

@interface ViewController ()

@property (strong, nonatomic) IBOutlet CBPeripheralManager *peripheralManager;
@property (strong, nonatomic) IBOutlet NSUUID *proximityUUID;

@end

// 省略

- (void)viewDidLoad
{
	[super viewDidLoad];
			    
	self.title = @"Peipheral";
					    
	self.proximityUUID = [[NSUUID alloc] initWithUUIDString:@"8D4DB809-032F-4771-96F3-99BD5C25F924"];
	self.peripheralManager = [[CBPeripheralManager alloc] initWithDelegate:self queue:nil options:nil];
	if (self.peripheralManager.state == CBPeripheralManagerStatePoweredOn) {
		[self startAdvertising];
	}
}

- (void)startAdvertising
{
	CLBeaconRegion *beaconRegion = [[CLBeaconRegion alloc] initWithProximityUUID: self.proximityUUID
																		   major:1
																		   minor:2
																	  identifier:@"com.test.ibeaconSample"];
	NSDictionary *beaconPeripheralData = [beaconRegion peripheralDataWithMeasuredPower:nil];
	[self.peripheralManager startAdvertising:beaconPeripheralData];
}
```

次に、Beaconを検知するCentralのソースを書きます。  
Projectの作成手順はPeripheralとほぼ同じで、  
1: BeaconCentralという名称で新規Project作成  
2: UIViewControllerを追加(ViewControllerという名称で作成しました)  
3: CoreLocationライブラリを追加  
として下さい。  

ViewController.hのソース  
```objective-c
#import <UIKit/UIKit.h>
#import <CoreLocation/CoreLocation.h>

@interface ViewController : UIViewController<CLLocationManagerDelegate>

@end
```

ViewController.mのソース  
```objective-c
#import "ViewController.h"

@interface ViewController ()

@property (strong, nonatomic) IBOutlet CLLocationManager *locationManager;
@property (strong, nonatomic) IBOutlet NSUUID *proximityUUID;
@property (strong, nonatomic) IBOutlet CLBeaconRegion *beaconRegion;
@property (strong, nonatomic) IBOutlet CLBeacon *nearestBeacon;

@end

// initWithNibNameは省略

- (void)viewDidLoad
{
	[super viewDidLoad];
	
	if ([CLLocationManager isMonitoringAvailableForClass:[CLCircularRegion class]]) {
		self.locationManager = [CLLocationManager alloc] init];]
		self.locationManager.delegate = self;

		self.proximityUUID = [[NSUUID alloc] initWithUUIDString:@"8D4DB809-032F-4771-96F3-99BD5C25F924"];
		self.beaconRegion = [[CLBeaconRegion alloc] initWithProximityUUID: self.proximityUUID identifier:@"com.test.ibeaconSample"];
		[self.locationManager startMonitoringForRegion: self.beaconRegion];
	} else {
		NSLog(@"お使いの端末ではiBeaconを利用できません。");
	}
}

// 領域計測が開始した場合
- (void)locationManager:(CLLocationManager *)manager didStartMonitoringForRegion:(CLRegion *)region
{
	[self sendLocalNotificationForMessage:@"Start Monitoring Region"];
}

// 指定した領域に入った場合
- (void)locationManager:(CLLocationManager *)manager didEnterRegion:(CLRegion *)region
{
	[self sendLocalNotificationForMessage:@"Enter Region"];
		    
	if ([region isMemberOfClass:[CLBeaconRegion class]] && [CLLocationManager isRangingAvailable]) {
		[self.locationManager startRangingBeaconsInRegion:(CLBeaconRegion *)region];
	}
}

// 指定した領域から出た場合
- (void)locationManager:(CLLocationManager *)manager didExitRegion:(CLRegion *)region
{
	[self sendLocalNotificationForMessage:@"Exit Region"];
	
	if ([region isMemberOfClass:[CLBeaconRegion class]] && [CLLocationManager isRangingAvailable]) {
		[self.locationManager stopRangingBeaconsInRegion:(CLBeaconRegion *)region];
	}
}

// Beacon信号を検出した場合
- (void)locationManager:(CLLocationManager *)manager didRangeBeacons:(NSArray *)beacons inRegion:(CLBeaconRegion *)region
{
	if (beacons.count > 0) {
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

		NSString *msg = [[NSString alloc] initWithFormat:@"%f [m]", self.nearestBeacon.accuracy];
		[self sendLocalNotificationForMessage:msg];
	}
}

// 領域観測に失敗した場合
- (void)locationManager:(CLLocationManager *)manager monitoringDidFailForRegion:(CLRegion *)region withError:(NSError *)error
{
	[self sendLocalNotificationForMessage:@"Exit Region"];
}

// ローカルプッシュの処理
- (void)sendLocalNotificationForMessage:(NSString *)message
{
	UILocalNotification *localNotification = [UILocalNotification new];
	localNotification.alertBody = message;
	localNotification.fireDate = [NSDate date];
	localNotification.soundName = UILocalNotificationDefaultSoundName;
	[[UIApplication sharedApplication] scheduleLocalNotification:localNotification];
}
```

これでソースは完了です。  
ソースを下記にアップしましたのでダウンロードしたい方はぜひ...。  
[Centralソース](https://github.com/grandbig/ibeaconCentralSample)  
[Peripheralソース](https://github.com/grandbig/ibeaconPeripheralSample)  
まずはアプリをBackgroundで起動させているときの動作を見てみてください。  
動作確認時には必ずCentralを先に起動する必要があります。約5秒後にPeripheralを起動して下さい。Centralの方にローカルプッシュが届くはずです。  

続いて、アプリがBackground起動していない場合を見てみましょう。PeripheralアプリとCentralのアプリの両方を停止状態にします。(先ほどの状態から続けて実験する場合は、必ずCentralアプリ側に**Exit Region**というメッセージでローカルプッシュが届いてからにしてください。)  
さて、Peripheralアプリを起動しましょう。しばらくするとCentralアプリを起動していないにも関わらず、ローカルプッシュが届くはずです。そして、Beacon間の距離がどんどんローカルプッシュされてくることでしょう。  
![Centralアプリを起動していない状態でローカルプッシュが届く](/images/ibeacon_ios71_1.png)  
その後、Peripheralアプリを停止してみましょう。しばらくするとCentralアプリに**Exit Region**というローカルプッシュが届くはずです。それを最後にローカルプッシュは来なくなります。  
![Beaconの検知終了](/images/ibeacon_ios71_2.png)  

このように、確かにアプリをBackgroundで起動していなくてもBeaconを検知するようです。因みに、このときアプリがBackground起動しているのか確認したのですが、起動はしてないんですね〜。というかホームボタン2回タップでBackground起動しているアプリが見れるはずなのに、そこに出てないんですね〜。う〜ん。どういうことなんでしょうか？？  

まぁ、何はともあれ、iOS7.1からiBeaconの扱いが劇的に変わるってことは間違いないようです。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
