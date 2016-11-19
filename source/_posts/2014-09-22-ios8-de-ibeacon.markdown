---
layout: post
title: "iOS8でiBeaconを試してみよう！！"
date: 2014-09-22 21:52
comments: true
categories: ios ios8 iBeacon
---

###iOS8になって気をつけるべきところについて
本日はiOS8端末でiBeaconを利用するときの注意点について書きたいと思います。  
iOS8ではユーザの位置情報に関する許可の選択肢が増えました。それは、**『許可しない』/『このAppの使用中のみ許可』/『常に許可』**の3つです。(これまでは単純にON/OFFに切替のみでした。)もちろん、ユーザの選択肢が増えたということは開発者側でも新たな対応が必要になるということです。  

![iOS8端末での位置情報の許可選択画面](/images/ios8_location_1.png)  

<!--more-->

具体的な対応としては、**ユーザに許可を求めるメソッドを実装する**というものです。  

下記にViewController.mファイルのソースを記載します。  
```objective-c
#import "ViewController.h"

@interface ViewController ()

@property (strong, nonatomic) IBOutlet CLLocationManager *locationManager;
@property (strong, nonatomic) IBOutlet NSUUID *proximityUUID;
@property (strong, nonatomic) IBOutlet CLBeaconRegion *beaconRegion;
@property (strong, nonatomic) IBOutlet CLBeacon *nearestBeacon;
@property (strong, nonatomic) IBOutlet NSString *str;

@end

@implementation ViewController

<省略>

- (void)viewDidLoad
{
	[super viewDidLoad];

	if ([CLLocationManager isMonitoringAvailableForClass:[CLCircularRegion class]]) {
		self.locationManager = [[CLLocationManager alloc] init];
		self.locationManager.delegate = self;
		
		self.proximityUUID = [[NSUUID alloc] initWithUUIDString:@"8D4DB809-032F-4771-96F3-99BD5C25F924"];
		self.beaconRegion = [[CLBeaconRegion alloc] initWithProximityUUID: self.proximityUUID
                                                               identifier:@"com.kato.ibeaconSample"];

		if ([self.locationManager respondsToSelector:@selector(requestAlwaysAuthorization)]) {
			// requestAlwaysAuthorizationメソッドが利用できる場合(iOS8以上の場合)
			// 位置情報の取得許可を求めるメソッド
			[self.locationManager requestAlwaysAuthorization];
		} else {
			// requestAlwaysAuthorizationメソッドが利用できない場合(iOS8未満の場合)
			[self.locationManager startMonitoringForRegion: self.beaconRegion];
		}
	} else {
		// iBeaconが利用できない端末の場合
		NSLog(@"iBeaconを利用できません。");
	}
}

// ユーザの位置情報の許可状態を確認するメソッド
- (void)locationManager:(CLLocationManager *)manager didChangeAuthorizationStatus:(CLAuthorizationStatus)status
{
	if (status == kCLAuthorizationStatusNotDetermined) {
		// ユーザが位置情報の使用を許可していない
	} else if(status == kCLAuthorizationStatusAuthorizedAlways) {
		// ユーザが位置情報の使用を常に許可している場合
		[self.locationManager startMonitoringForRegion: self.beaconRegion];
	} else if(status == kCLAuthorizationStatusAuthorizedWhenInUse) {
		// ユーザが位置情報の使用を使用中のみ許可している場合
		[self.locationManager startMonitoringForRegion: self.beaconRegion];
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

// 領域内にいるかどうかを確認する処理
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

// Beacon信号を検出した場合
- (void)locationManager:(CLLocationManager *)manager didRangeBeacons:(NSArray *)beacons inRegion:(CLBeaconRegion *)region
{
	if (beacons.count > 0) {
		self.nearestBeacon = beacons.firstObject;
		self.str = [[NSString alloc] initWithFormat:@"%f [m]", self.nearestBeacon.accuracy];
		NSLog(@"%@", self.str);
		[self sendLocalNotificationForMessage:self.str];
	}
}

// ローカルプッシュ処理
- (void)sendLocalNotificationForMessage:(NSString *)message
{
	UILocalNotification *localNotification = [UILocalNotification new];
	localNotification.alertBody = message;
	localNotification.fireDate = [NSDate date];
	localNotification.soundName = UILocalNotificationDefaultSoundName;
	[UIApplication sharedApplication] scheduleLocalNotification:localNotification];]
}

```

これにより、アプリを起動した際に、ユーザへ位置情報の使用を求めるアラートを表示することができます。  
![ユーザへのアラート](/images/ios8_location_2.png)  

気をつけたいのが、iOS8からは**Info.plistファイルに位置情報を使用する理由**を記載する必要があります。  
Info.plistにNSLocationWhenInUseUsageDescriptionまたはNSLocationAlwaysUsageDescriptionを設定する必要があります。  
ただし、Info.plistでAdd Rowしようとしても、候補に表示されないので、手打ちで設定しましょう。  

###iOS8でWhenInUseとAlwaysの挙動の違い
続いて、iOS8から追加されたWhenInUse(このAppの使用中のみ許可)とAlways(常に許可)でiBeaconの挙動にどういった差が出るか調べてみたいと思います。  
まずは、Always(常に許可)から説明します。  
こちらは、領域観測(didEnterRegion/didDetermineState/didExitRegion)と距離計測(didRangeBeacons)の両方を実行できます。  
※基本的にはiOS7.1.xと同じ挙動と言えます。  

一方でWhenInUseを見てみると、距離計測のみが可能でした。  
領域観測ができないため、startRangingBeaconsInRegionメソッドを実行するタイミングをミスらないように注意が必要です。  
例えば、**didEnterRegion** Delegateメソッドの中でstartRangingBeaconsInRegionメソッドを実行とすると、WhenInUseではiBeaconを一切検知できないようになってしまいます。  
アプリをフォアグラウンドに起動したときだけ、iBeaconの機能を利用したいのであれば、確かにWhenInUseで十分だと思います。  

###ローカルプッシュの補足
因みに、筆者は上記の動作を確認するためにローカルプッシュ処理を組み込んだのですが、そこでハマりました。なぜかというと、iOS8ではローカルプッシュがうまくいかなかったからです。  
いろいろと調べた結果、iOS8からは**ローカルプッシュの設定を記載する**必要があるとのことでした。  

以下のようにAppDelegate.mファイルに記載すれば、ローカルプッシュが利用可能になります。  
```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
	<省略>

	if ([UIApplication instancesRespondToSelector:@selector(registerUserNotificationSettings:)]) {
		[[UIApplication sharedApplication] registerUserNotificationSettings:[UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeAlert|UIUserNotificationTypeSound categories:nil]];
	}

	return YES;
}
```

一先ず、ここまで。  

参考:  
[iOS8から位置情報を取得する方法が変わるよ](http://qiita.com/koogawa/items/6ec24ca3201977e9642b)  
[iOS8の位置情報のプライバシー設定に対応する](http://im-sei.tumblr.com/post/91824653043/ios-8)  
[iOS8からローカル通知が保存されなくなった？](http://qiita.com/caesar_cat/items/08018ab22bea27b55443)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

