---
layout: post
title: "iOS8.1でCLVisitとCMPedometerを試してみた！"
date: 2014-11-30 17:27
comments: true
categories: ios ios8 CLVisit CMPedometer 
---

####iOS8から追加された新しい機能 CLVisitとCMPedometer
さて、久々の更新です。  
今更ですが、CLVisitとCMPedometerについて調べてみました。  
CLVisitはiOS8以降、CMPedometerはiOS8以降かつM8 モーションコプロセッサを搭載したiPhone6 or iPhone6 Plusでのみ利用可能です。  
iOS8はともかく、まだまだ、iPhone6 or iPhone6 Plusは普及しているとは言い難いことでしょう。  
実質的に多いのはiPhone5かもしれません...。  
しかし、ここはエンジニアとして、1〜2年後を見て、アプリを開発すべきでしょう。  
ということで早速、見て行きましょう。  

<!-- more -->

####CLVisitで滞在時間を検知
まずは、CLVisitから見ていきます。  
CLVisitは特定の場所の滞在時間を取得できるクラスです。  
最近、Swiftでソースを書いてあるサイトが多いので、あえてObjective-Cで書いてみます。  

```objective-c
// ViewController.m

#import "ViewController.h"
#import <CoreLocation/CoreLocation.h>

@interface ViewController () <CLLocationManagerDelegate>

@property (strong, nonatomic) CLLocationManager *lm;
@property (weak, nonatomic) IBOutlet UILabel *label1;
@property (weak, nonatomic) IBOutlet UITextView *textView1;

@end

@implementation ViewController

- (void)viewDidLoad {

	[super viewDidLoad];

	self.lm = [[CLLocationManager alloc] init];
	self.lm.delegate = self;
	
	// 位置情報の取得許可を求めるメソッド
	[self.lm requestAlwaysAuthorization];
}

- (void)didReceiveMemoryWarning {
	[super didReceiveMemoryWarning];
}

- (void)locationManager:(CLLocationManager *)manager didChangeAuthorizationStatus:(CLAuthorizationStatus)status {
	
	if (status == kCLAuthorizationStatusNotDetermined) {
		// ユーザが位置情報の使用を許可していない
	} else if(status == kCLAuthorizationStatusAuthorizedAlways) {
		// ユーザが位置情報の使用を常に許可している場合
		// 滞在時間の取得開始
		[self.lm startMonitoringVisits];
	} else if(status == kCLAuthorizationStatusAuthorizedWhenInUse) {
		// ユーザが位置情報の使用を使用中のみ許可している場合
	}
}

- (void)locationManager:(CLLocationManager *)manager didVisit:(CLVisit *)visit {
	
	NSDateFormatter *outputFormatter = [[NSDateFormatter alloc] init];
	[outputFormatter setDateFormat:@"yyyy-MM-dd HH:mm:ss"];

	NSString *arrivalDate = [outputFormatter stringFromDate:visit.arrivalDate];
	NSString *departureDate = [outputFormatter stringFromDate:visit.departureDate];
	NSString *latitude = [NSString stringWithFormat:@"%f",visit.coordinate.latitude];
	NSString *longitude = [NSString stringWithFormat:@"%f",visit.coordinate.longitude];

	NSString *message = [[NSString alloc] initWithFormat:@"緯度: %@\n経度: %@\n到着時間: %@\n出発時間: %@", latitude, longitude, arrivalDate, departureDate];
	// ローカルプッシュ通知
	[self sendLocalNotificationForMessage:message soundFlag:NO];
	dispatch_async(dispatch_get_main_queue(), ^{
		self.textView1.text = [[NSString alloc] initWithFormat: message];
	});
}

/**
 ローカルプッシュ通知処理
 @param message メッセージ
 @param sound 通知音の設定
 */
- (void)sendLocalNotificationForMessage:(NSString *)message soundFlag:(BOOL)soundFlag {
	
	UILocalNotification *localNotification = [UILocalNotification new];
	localNotification.alertBody = message;
	localNotification.fireDate = [NSDate date];
	if(soundFlag) {
		localNotification.soundName = UILocalNotificationDefaultSoundName;
	}
	[[UIApplication sharedApplication] scheduleLocalNotification:localNotification];
}

@end

```

このアプリをインストールしたiPhone6を持って、ぶらぶらしてみたところ...  

緯度: 35.☓☓☓☓☓☓  
経度: 139.☓☓☓☓☓  
到着時間: 2014-11-30 16:00:09  
出発時間: 2014-11-30 16:22:20  

といった結果が得られました。  
その場を離れてから5〜6分後にローカルプッシュ通知が来ました。  
※トリガーが時間なのか位置情報なのかは未検証  

因みに、 **アプリを1度起動すれば、その後、Backgroundで起動していなくても、検知してくれます。**   
これは滞在時間の取得以外にも何かに使えそうですね。  
また、移動中は検知しなかったので、滞在と見なす時間が決まっているのでしょう。  


####CMPedometerで歩行情報を取得
次に、CMPedometerを見ていきます。  
CMPedometerはiOS7で追加されていたCMStepCounterのパワーアップバージョンです。  
これもObjective-Cでソースを書いてみます。  

```objective-c
// ViewController.m

#import "ViewController.h"
#import <CoreMotion/CoreMotion.h>

@interface ViewController ()

@property (strong, nonatomic) CMPedometer *pmp;
@property (weak, nonatomic) IBOutlet UILabel *label1;
@property (weak, nonatomic) IBOutlet UITextView *textView1;

@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];

	self.pmp = [[CMPedometer alloc] init];
	BOOL check = [self confirmCMPedometer];
	if(check) {
		// モーション計測の開始
		[self startPedometer];
	}
}

- (void)didReceiveMemoryWarning {
	[super didReceiveMemoryWarning];
}

/**
 CMPedometerが利用可能か判断する処理
 return 判断結果(YES: 利用可能, NO: 利用不可能)
 */
- (BOOL)confirmCMPedometer {
	if([CMPedometer isStepCountingAvailable] && [CMPedometer isDistanceAvailable] && [CMPedometer isFloorCountingAvailable]) {
		return YES;
	} else {
		return NO;
	}
}

/**
 歩行動作を計測する処理
 */
- (void)startPedometer {
	
	[self.pmp startPedometerUpdatesFromDate:[NSDate date]
	                            withHandler:^(CMPedometerData *pedometerData, NSError *error) {
									dispatch_async(dispatch_get_main_queue(), ^{
										// step数の取得
										NSNumber *step = pedometerData.numberOfSteps;
										// 距離
										NSNumber *distance = pedometerData.distance;

										// 日時取得のためにFormatを設定
										NSDateFormatter *outputFormatter = [[NSDateFormatter alloc] init];
										[outputFormatter setDateFormat:@"yyyy-MM-dd HH:mm:ss"];
										// 開始日時
										NSString* startDate = [outputFormatter stringFromDate:pedometerData.startDate];
										//終了日時
										NSString *endDate = [outputFormatter stringFromDate:pedometerData.endDate];

										// 階段の昇降
										NSNumber *floorsAscended = pedometerData.floorsAscended;
										NSNumber *floorsDescended = pedometerData.floorsDescended;

										NSString *str = [[NSString alloc] initWithFormat:@"Step Count Result\n\n歩数: %@\n距離: %@[m]\n開始時間: %@\n終了時間: %@\n上った回数: %@\n下りた回数: %@", step, distance, startDate, endDate, floorsAscended, floorsDescended];
										self.textView1.text = str;

										// ローカルプッシュ通知
										[self sendLocalNotificationForMessage:str soundFlag:NO];
									});

	}];
}

/**
 ローカルプッシュ通知処理
 @param message メッセージ
 @param sound 通知音の設定
 */
- (void)sendLocalNotificationForMessage:(NSString *)message soundFlag:(BOOL)soundFlag {
	
	UILocalNotification *localNotification = [UILocalNotification new];
	localNotification.alertBody = message;
	localNotification.fireDate = [NSDate date];
	if(soundFlag) {
		localNotification.soundName = UILocalNotificationDefaultSoundName;
	}
	[[UIApplication sharedApplication] scheduleLocalNotification:localNotification];
}

@end

```

このアプリをインストールしたiPhone6を持って、ぶらぶらしてみたところ...  

歩数: 1128  
距離: 774.0900000002002 [m]  
開始時間: 2014-11-30 15:57:22  
終了時間: 2014-11-30 16:29:32  
上がった回数: 2  
下がった回数: 0  

といった結果が得られました。  
これはローカルプッシュ通知が届くタイミングが任意なのか、法則性はなさそうでした。  
※トリガーが時間なのか、歩数なのか、距離なのか未検証  

因みに、 アプリを1度起動すれば、その後、Backgroundで起動していなくても、検知してくれます。  
これは歩数計測以外にも何かに使えそうですね。  

ということで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
