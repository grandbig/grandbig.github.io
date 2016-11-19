---
layout: post
title: "複数のiBeaconを検知しよう！(別UUIDの場合)"
date: 2015-06-14 23:43
comments: true
categories: ios iBeacon
---

####複数のiBeacon(別UUIDのiBeacon)を検知する方法
さて、本日は別々のUUIDを持ったiBeacon信号を補足したいと思います。  
iBeaconは案外、電波が遠くまで飛ぶため、幾つかのUUIDを補足できるようにしておくことで、うまく`didEnterRegion`, `didExitRegion`を使い、レンジングでのみ取得可能な値を効果的に使うことができるかもしれません。  
今回はiOS8以上を対象に下記サンプルソースを書きました。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

```objective-c
// ViewController.m
#import "ViewController.h"

@interface ViewController ()

@property (strong, nonatomic) CLLocationManager *locationManager;
@property (strong, nonatomic) NSUUID *proximityUUID1, *proximityUUID2;
@property (strong, nonatomic) CLBeaconRegion *beaconRegion1, *beaconRegion2;
@property (strong, nonatomic) CLBeacon *nearestBeacon;
@property (strong, nonatomic) NSString *str;

@end

- (void)viewDidLoad
{
	[super viewDidLoad];
	
	self.locationManager = [[CLLocationManager alloc] init];
	self.locationManager.delegate = self;

	self.proximityUUID1 = [[NSUUID alloc] initWithUUIDString:@"8D4DB809-032F-4771-96F3-99BD5C25F924"];
	self.beaconRegion1 = [[CLBeaconRegion alloc] initWithProximityUUID: self.proximityUUID1 identifier:@"com.takahiro.ibeaconSample"];
	self.proximityUUID2 = [[NSUUID alloc] initWithUUIDString:@"1AE93327-B172-4C5D-BBF7-F52B959FD4EB"];
	self.beaconRegion2 = [[CLBeaconRegion alloc] initWithProximityUUID: self.proximityUUID1 identifier:@"com.takahiro.ibeaconSample2"];

	[self.locationManager requestAlwaysAuthorization];
}

// ユーザの位置情報の許可状態を確認するメソッド
- (void)locationManager:(CLLocationManager *)manager didChangeAuthorizationStatus:(CLAuthorizationStatus)status
{
	if (status == kCLAuthorizationStatusAuthorizedAlways) {
		// Beaconのモニタリングを開始
		[self.locationManager startMonitoringForRegion:self.beaconRegion1];
		[self.locationManager startMonitoringForRegion:self.beaconRegion2];
	}
}

// 指定した領域に入った場合
- (void)locationManager:(CLLocationManager *)manager didEnterRegion:(CLRegion *)region
{
	[self sendLocalNotificationForMessage:@"Enter Region"];
}

// 指定した領域から出た場合
- (void)locationManager:(CLLocationManager *)manager didExitRegion:(CLRegion *)region
{
	[self sendLocalNotificationForMessage:@"Exit Region"];
}

// iBeacon領域内に既にいるか/いないかの判定
- (void)locationManager:(CLLocationManager *)manager didDetermineState:(CLRegionState)state forRegion:(CLRegion *)region
{
	switch (state) {
		case CLRegionStateInside:
			if([region isMemberOfClass:[CLBeaconRegion class]] && [CLLocationManager isRangingAvailable]){
				[self.locationManager startRangingBeaconsInRegion:self.beaconRegion1];
				[self.locationManager startRangingBeaconsInRegion:self.beaconRegion2];
			}
			break;
		case CLRegionStateOutside:
			if([region isMemberOfClass:[CLBeaconRegion class]] && [CLLocationManager isRangingAvailable]){
			}
			break;
		case CLRegionStateUnknown:
			if([region isMemberOfClass:[CLBeaconRegion class]] && [CLLocationManager isRangingAvailable]){
			}
			break;
		default:
			break;
	}
}

// Beacon信号を検出した場合
- (void)locationManager:(CLLocationManager *)manager didRangeBeacons:(NSArray *)beacons inRegion:(CLBeaconRegion *)region
{
	if (beacons.count > 0) {
		self.nearestBeacon = beacons.firstObject;
		NSString *rangeMessage;

		switch(self.nearestBeacon.proximity) {
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

// ローカルプッシュ
- (void)sendLocalNotificationForMessage:(NSString *)message
{
	UILocalNotification *localNotification = [UILocalNotification new];
	localNotification.alertBody = message;
	localNotification.fireDate = [NSDate date];
	localNotification.soundName = nil;
	[[UIApplication sharedApplication] scheduleLocalNotification:localNotification];
}
```

ローカルプッシュを可能にするために`AppDelegate.m`に下記書式を追加しましょう。  

```objective-c
// AppDelegate.m
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
	...

	if ([UIApplication instancesRespondToSelector:@selector(registerUserNotificationSettings:)]) {
		[[UIApplication sharedApplication] registerUserNotificationSettings:[UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeAlert|UIUserNotificationTypeSound categories:nil]];
	}

	return YES;
}
```

ポイントは`CLBeaconRegion`を作成するときに指定する`UUID`と`identifier`が共にユニークであることです。  
`UUID`のみを別々にしても`identifier`が同じだと別のiBeacon領域と見なされないので注意が必要です。  

別々のiBeacon領域が作成できれば、`didEnterRegion`, `didExitRegion`などのデリゲートメソッドはそれぞれのiBeacon領域に対して発生します。  
もし、比較的狭い空間で複数のiBeaconを設置して、アプリをFG起動しない前提で人流解析をしたい場合は複数のiBeacon領域(別々のUUIDを持ったiBeacon領域)を用意することも１つの手だと思います。  

と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
