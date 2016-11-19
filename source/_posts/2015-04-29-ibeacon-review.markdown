---
layout: post
title: "iBeacon完全攻略！？"
date: 2015-04-29 21:51
comments: true
categories: ios iBeacon
---

####iBeaconを復習しよう！
本日はこれまでも何度か取り上げてきたiBeaconについて改めて復習してみたいと思います。  
今回はiOS端末でのiBeaconの発信/受信に特化して書きます。  

これまでの関連記事は以下です。  

* [噂のiOS7.1でiBeaconを試してみよう！！](http://grandbig.github.io/blog/2014/04/12/io7-dot-1-de-ibeacon/)
* [iOS8でiBeaconを試してみよう！！](http://grandbig.github.io/blog/2014/09/22/ios8-de-ibeacon/)
* [Swiftを使って、iBeaconのCentralアプリを作ろう！！](http://grandbig.github.io/blog/2014/09/29/developing-ibeacon-app-by-using-swift/)
* [複数のiBeacon信号を利用してみよう！](http://grandbig.github.io/blog/2015/04/13/multi-ibeacon/)

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

では、早速、まとめていきましょう。  

####iBeaconが利用可能な端末

* OS: iOS7.0以上
* 端末: iPhone4S以降, iPad(第３世代)以降, iPad mini以降, iPod touch(第５世代以降)
※もちろんiPad Airでも利用可能

####iOSごとのiBeacon機能の差異

まずは動作面での差異について  

* iOS7.0.xの場合  
アプリをFG起動もしくはBG起動していないとiBeaconを検知することはできない
* iOS7.1.x以降の場合  
アプリを起動していない(停止状態の)場合でもiBeaconを検知可能

次にプログラミング面での差異について  

プログラミング面ではCentral側にのみ多少の差異があります。  
iBeaconの検知には **CoreLocation.framework** を利用します。
**CoreLocation.framework** は位置情報サービスを利用するためのフレームワークですが、iBeaconの領域監視メソッドが組み込まれています。  
もともと、ジオフェンスの領域監視メソッドが組み込まれていますので、それと同等に扱いたいというApple側の意図が見えます。  

具体的な差異について説明します。  
位置情報サービスの利用許可メソッドをiOS8から組み込む必要があります。  

```objective-c
#import "ViewController.h"

@interface ViewController ()

@property (strong, nonatomic) CLLocationManager *lm;
@property (strong, nonatomic) NSUUID *proximityUUID;
@property (strong, nonatomic) CLBeaconRegion *beaconRegion;
@property (strong, nonatomic) CLBeacon *nearestBeacon;
@property (strong, nonatomic) NSString *str;

@end

@implementation ViewController

- (void)viewDidLoad
{
	[super viewDidLoad];

	if ([CLLocationManager isMonitoringAvailableForClass:[CLCircularRegion class]]) {
		// iOS7.x以降の端末
		// CLLocationManagerの初期化
		self.lm = [[CLLocationManager alloc] init];
		self.lm.delegate = self;

		// UUIDの設定
		self.proximityUUID = [[NSUUID alloc] initWithUUIDString:@"8D4DB809-032F-4771-96F3-99BD5C25F924"];
		// Bundle Identifierの取得
		NSBundle *bundle = [NSBundle mainBundle];
		NSString *bid = [bundle bundleIdentifier];
		self.beaconRegion = [[CLBeaconRegion alloc] initWithProximityUUID: self.proximityUUID identifier: bid];

		if ([self.locationManager respondsToSelector:@selector(requestAlwaysAuthorization)]) {
			// iOS8以上の場合
			// 位置情報サービスを常に許可させて良いかどうかを尋ねるためのメソッド
			[self.lm requestAlwaysAuthorization];
		} else {
			// iOS8未満の場合
			// iBeacon領域の監視を開始
			[self.lm startMonitoringForRegion: self.beaconRegion];
		}
	} else {
		// iOS6.x以前の端末
		UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"確認" message:@"iBeacon機能を利用できない端末です。" delegate: nil cancelButtonTitle: nil otherButtonTitles:@"OK", nil];
		// アラートを表示
		[alert show];
	}
}

// ユーザの位置情報の許可状態を確認するメソッド
- (void)locationManager:(CLLocationManager *)manager didChangeAuthorizationStatus:(CLAuthorizationStatus)status
{
	if (status == kCLAuthorizationStatusAuthorizedAlways) {
		// ユーザが位置情報の使用を常に許可している場合
		// iBeacon領域の監視を開始
		[self.lm startMonitoringForRegion: self.beaconRegion];
	} else {
		// その他の場合
		// 設定画面に遷移
		NSURL *url = [NSURL URLWithString:UIApplicationOpenSettingsURLString];
		[[UIApplication sharedApplication] openURL:url];
	}
}

@end
```

iOS7.xでは`startMonitoringForRegion`メソッドを実行した段階で`didChangeAuthorizationStatus`メソッドが呼ばれます。  
iOS8では、`requestAlwaysAuthorization`メソッドを実行(位置情報サービスを常に許可する場合のメソッド)しなければ、`didChangeAuthorizationStatus`メソッドが呼ばれないため、`startMonitoringForRegion`メソッドの実行タイミングを変える必要が出てきました。  

####iBeacon関連メソッドの実行順について(Centralの場合)
先ほどiBeaconの検知は **CoreLocation.framework** 内のメソッドとして組み込まれていると説明しました。では、他にどんなメソッドがあるのでしょうか。  

```objective-c
// 領域監視が開始された後に呼び出される処理
- (void)locationManager:(CLLocationManager *)manager didStartMonitoringForRegion:(CLRegion *)region {
}
```

```objective-c
// 領域監視に失敗した場合に呼び出される処理
- (void)locationManager:(CLLocationManager *)manager monitoringDidFailForRegion:(CLRegion *)region withError:(NSError *)error {
}
```

```objective-c
// 監視領域に入った場合に呼び出される処理
- (void)locationManager:(CLLocationManager *)manager didEnterRegion:(CLRegion *)region {
}
```

```objective-c
// 監視領域から出た場合に呼び出される処理
- (void)locationManager:(CLLocationManager *)manager didExitRegion:(CLRegion *)region {
}
```

```objective-c
// 監視領域に対する状態が変化した場合に呼び出される処理
- (void)locationManager:(CLLocationManager *)manager didDetermineState:(CLRegionState)state forRegion:(CLRegion *)region {
}
```

```objective-c
// iBeacon信号をレンジング検知した場合に呼び出される処理
- (void)locationManager:(CLLocationManager *)manager didRangeBeacons:(NSArray *)beacons inRegion:(CLBeaconRegion *)region {
}
```

上記のメソッドを使えば十分なアプリが開発できるはずです。  
さて、ここで注意しておくべきなのは、`startRangingBeaconsInRegion`メソッドをどのタイミングで実行するかということです。  

基本的には、 **監視領域に入ったタイミング** で`startRangingBeaconsInRegion`メソッドを実行するのが普通かもしれません。  
しかし、この場合は注意が必要です。  
なぜなら、監視領域内でアプリを落として、再度起動した場合、`didEnterRegion`メソッドが実行されないからです。つまり、既に監視領域に入っている場合は **監視領域に侵入したと見なされない** ということです。  
よって、こういったケースがアプリの利用に打撃を与えるのであれば、 **監視領域に入ったタイミング** のみに`startRangingBeaconsInRegion`メソッドを置くわけにはいかないことになります。  
その場合は、`didDetermineState`メソッドで既に監視領域内にいる場合(`state`が`CLRegionStateInside`の場合)に`startRangingBeaconsInRegion`メソッドを実行するようにしましょう。  

####Peripheralの注意点
続いて、PeripheralとしてiOS端末を使う場合の注意点についてお話しておきます。  
iOS端末ではアプリをFG起動している間のみiBeacon信号の発信が可能です。  
アプリがBG起動になった場合、iBeacon信号の発信が止まってしまうため、数秒後にCentral側のiOS端末で`didExitRegion`メソッドが実行されます。  
なので、iOS端末でPeripheralの役割を担いたいのであれば、FG起動を続けることに問題がない使い方である場合に限ります。  

因みに、XcodeのBackground Modesで **Act as a Bluetooth LE accessory** を有効にしたとしても、BG起動中はiBeaconを発信することはできません。  

また、 **CoreBluetooth.framework** にはiOS端末がiBeacon発信状態(アドバタイジング状態)かを判別する`isAdvertising` プロパティが存在します。  
iBeaconを発信している状態でアプリをBG起動にした場合は`isAdvertising`は **YES** として返却されます。つまり、実際の状態と必ずしも一致するわけではないということです。  

####Centralの注意点
最後にCentralとしてiOS端末を使う場合の注意点についてお話しておきます。  
iOS端末でiBeaconのレンジング処理を行う場合、FG起動時にしかレンジング処理を実行できません。  
そのため、アプリの状態によらずレンジング処理が必要な仕様を実現することは不可能ということになります。  

一応、`didEnterRegion`メソッドや`didExitRegion`メソッドなどのデリゲートメソッドが実行された場合、約10秒間はアプリの状態によらずあらゆる処理が実行可能であるため、この間のみレンジング処理を実行して`major`, `minor`などの値を取得することは可能です。  

正しい動作検証を実施した上でiBeaconを扱うようにしましょう。  

####各種設定がOFFの場合のアラート表示について
さて、直接iBeaconとは関係がありませんが、iOS端末をPeripheralとして扱う場合は **Bluetoothの設定** をONにしておく必要があります。iOS端末をCentralとして扱う場合は **Bluetoothの設定** と **位置情報サービスの設定** をONにしておく必要があります。  

これらがOFFになっている場合、iBeaconの機能を利用することができないため、アプリ開発時にアラートを表示してユーザに知らせることを考えるかと思います。  
さらに、できれば設定画面に飛ばしたいと思いますよね？  

**位置情報サービス** の場合は、iOS8であれば設定画面へのURLスキームが復活したため、問題ありません。(iOS7.xでは設定画面への遷移は諦めましょう。)  
**Bluetooth** の場合は **CoreBluetooth.framework** を利用していれば、難しくありません。(処理に **CoreBluetooth.framework** が不要なCentral側であっても設定画面に飛ばしたいのであれば、`import`する必要があります。)  
※ 具体的には`CBPeripheralManager`もしくは`CBCentralManager`の初期化時に`option`に`CBPeripheralManagerOptionShowPowerAlertKey`もしくは`CBCentralManagerOptionShowPowerAlertKey`を設定すれば良いです。  

以上がまとめとなります。  
ぜひぜひ参考にして頂ければと思います。  
本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
