---
layout: post
title: "Backgroundで位置情報を取得しよう！"
date: 2013-09-27 22:54
comments: true
categories: ios location
---

###アプリがBackground起動のときでも位置情報を取得しよう！
本日は位置情報を使ったアプリの開発について紹介します。  
皆さんはiPhoneを使うとき果たして位置情報サービスをONにされてますでしょうか？  
スマートフォンが多くの人に普及した近年では、その電池使用量の多さから『電池の使用を抑えるためには位置情報サービスをOFFにしましょう！』と言った声が現実にもネットにもあふれんばかりです。確かに位置情報サービスをONにしているとGPSを使うため、通常よりも電池を消費してしまいます。しかしながら、それ以上に非常に便利で役立つお得なアプリがどんどん世の中に出てきています。また、iOS7からiBeaconをサポートするため、その勢いは留まることを知らないでしょう。  
そう、今後は位置情報サービスをONにすることへの抵抗が下がっていく可能性があるのです。では早速、開発について話を進めていきましょう！  

<!--more-->

####簡単に位置情報を習得してみよう！
まず、以前紹介した[Let's Create an iPhone App](http://grandbig.github.io/blog/2013/09/13/lets-create-an-iphone-app/)のときと同様に下記状態までプロジェクト作成を進めて下さい。  
![LocationPractice.xcodeprojを作成](/images/locationpractice.png)  
『Let's Create an iPhone App』のとき作成したようにViewControllerを追加し、今回は**CoreLocation.framework**というライブラリを追加しましょう。  

位置情報の取得をするためにはCoreLocation.frameworkの中で定義されている**CLLocationManagerクラス**を利用します。  
ViewController.mファイル内でCLLocationManagerクラスを使うためにViewController.hファイルで以下を定義しましょう。  

```
// ViewController.hファイルの内容

#import <UIKit/UIKit.h>
// ①CoreLocationのimportを書く
#import <CoreLocation/CoreLocation.h>

// ②DelegateにCLLocationManagerDelegateを設定
@interface ViewController : UIViewController <CLLocationManagerDelegate> {
	// ③メンバ変数を定義
	IBOutlet CLLocationManager *locationManager;
}
```
①CoreLocation.frameworkをViewController.m内で利用するためにimportする  
②CLLocationManager関係の処理はCLLocationManagerクラスに任せてしまいたいのでDelegateを設定  
※深く考えこまずにframwork使うときはDelegateを設定するようにしましょう。  
③メンバ変数としてCLLocationManagerを定義します。  

次にViewController.mファイルを編集します。
```
// ViewController.mファイルの内容

...
- (void)viewDidLoad {
	[super viewDidLoad];
	// ①locationManager初期化
	locationManager = [[CLLocationManager alloc] init];
	// ②位置情報サービスのON/OFFで挙動を分岐
	if ([CLLocationManager locationServicesEnabled]) {
		// ③locationManagerの各プロパティを設定
		locationManager.delegate = self;
		locationManager.desiredAccuracy = kCLLocationAccuracyHundredMeters;
		locationManager.activityType = CLActivityTypeFitness;
		locationManager.pausesLocationUpdatesAutomatically = NO;
		locationManager.distanceFilter = 100.0;
		// ④位置情報の取得開始
		[locationManager startUpdatingLocation];
	} else {
		NSLog(@"Location services not available.");
	}
}
...
// ⑤位置情報更新時
- (void)locationManager:(CLLocationManager *)manager
    didUpdateToLocation:(CLLocation *)newLocation
	           fromLocation:(CLLocation *)oldLocation {
	// ⑥ログを出力
	NSLog(@"didUpdateToLocation latitude=%f, longitude=%f, accuracy=%f, time=%@",
		[newLocation coordinate].latitude,
		[newLocation coordinate].longitude,
		newLocation.horizontalAccuracy,
		newLocation.timestamp);
}
```
①当然のごとく初期化  
②位置情報サービスの設定がON/OFFの場合で挙動を分けるために条件分岐を用意  
③locationManagerの各プロパティを設定しましょう。  
・desireAccuracy：位置情報の取得精度の設定です。6種類の中から選択します。  
　kCLLocationAccuracyBestForNavigation：iOS4以降から使用可能です。最高精度。  
　kCLLocationAccuracyBest：iOS3までの最高精度設定でした。  
　kCLLocationAccuracyNearestTenMeters：誤差10mの設定  
　kCLLocationAccuracyHundredMeters：誤差100mの設定  
　kCLLocationAccuracyKilometer：誤差1kmの設定  
　kCLLocationAccuracyThreeKilometers：誤差3kmの設定  
・activityType：ユーザの移動タイプに合わせて位置情報の更新頻度を設定可能です。(iOS6以降から使用可能)  
　CLActivityTypeFitness：ユーザが歩行移動のときに最適  
　CLActivityTypeAutomotiveNavigation：ユーザが車で移動するときに最適  
　CLActivityTypeOtherNavigation：ユーザがボート/電車/飛行機で移動するときに最適  
　CLActivityTypeOther：その他  
・pausesLocationUpdatesAutomatically：位置情報が自動的にOFFになる設定(iOS6以降から使用可能)  
　アプリがBackground起動中に位置情報の更新が15分以上ない場合に自動でGPS起動がOFFになります。NOを設定することで回避できます。  
・distanceFilter：ここで設定した距離以上移動した場合に位置情報を取得する設定  
　単位は[m]です。  
④位置情報の取得を開始します。  
　停止する場合はstopUpdatingLocationになります。  
⑤位置情報が更新されたときに実行される関数です。  
⑥位置情報が正しく取得されていることを確認するためにログを出力しましょう。  
　latitude, longitude: 緯度, 経度  
　verticalAccuracy: 海抜高度の精度  
　horizontalAccuracy：水平精度(マップを見たときの縦横の精度)    
　timestamp：位置情報の取得時刻  
　speed：速度  
　cource：方角  

たったこれだけのことでiPhoneの位置情報を取得することができるようになります。  


####NSTimerで定期時間ごとに位置情報を取得してみよう！
先ほどdidUpdateToLocationを紹介しました。これは位置情報が更新された際に実行されます。逆に言えば、いつ位置情報を使った処理を実行できるかがわからないということになります。もし、5分毎に位置情報を使った処理を実行したいということがあればNSTimerを使う必要があります。  
それではNSTimerを使って先ほどのソースを書き直してみます。  

```
// ViewController.hファイルの内容

#import <UIKit/UIKit.h>
#import <CoreLocation/CoreLocation.h>

@interface ViewController : UIViewController <CLLocationManagerDelegate> {
	IBOutlet CLLocationManager *locationManager;
	// NSTimerを定義
	IBOutlet NSTimer *timer;
}
```

```
// ViewController.mファイルの内容
...
- (void)viewDidLoad {
	[super viewDidLoad];
	locationManager = [[CLLocationManager alloc] init];
	if ([CLLocationManager locationServicesEnabled]) {
		locationManager.delegate = self;
		locationManager.desiredAccuracy = kCLLocationAccuracyHundredMeters;
		locationManager.activityType = CLActivityTypeFitness;
		locationManager.pausesLocationUpdatesAutomatically = NO;
		locationManager.distanceFilter = 100.0;
		[locationManager startUpdatingLocation];
		// 5分ごとに繰り返しgetGpsDataを実行するようにタイマーを設定
		timer = [NSTimer scheduledTimerWithTimeInterval:300 target:self selector:@selector(getGpsData:)userInfo:nil repeats:YES];
	} else {
		NSLog(@"Location services not available.");
	}
}
...
- (void)getGpsData:(NSTimer *)theTimer {
	CLLocation *location = [locationManager location];
	CLLocationCoordinate2D coordinate = [location coordinate];// 座標を取得
	NSString *lat = [NSString alloc] initWithFormat:@"%f", coordinate.latitude];  // 経度を取得
	NSString *lng = [NSString alloc] initWithFormat:@"%f", coordinate.longitude]; // 緯度を取得
	NSLog(@"+++++ [デリゲートによらない場合]緯度,经度: %@, %@", lat, lng);
	[lat release];
	[lng release];
}
```

####Backgroundでも位置情報サービスを使えるように設定しよう！
せっかくNSTimerを使って、定期間隔ごとに位置情報を利用した処理を実行することができるようになったので、アプリをBackground起動したときにも処理が実行されるように設定していきましょう。  
①プロジェクトのCapabilityのBackgrounds Modeを設定(Xcode5から設定が必要)  
プロジェクト > ターゲット > Capability > Background Modeの**Location updates**にチェックを入れる  
![Location updatesにチェック](/images/locationpractice2.png)  
②plistの編集(どのXcodeのバージョンでも設定が必要)  
Add Rowで**Required Background Modes**を追加して下さい。次にその値として**App registers for location updates**を選択して下さい。  
![Required Background Modesを追加](/images/locationpractice3.png)  

これで端末をPCに繋いでプロジェクトを実行している状態で、ログの出力を確認して下さい。アプリがForeground起動のときでもBackground起動のときでもログが出力されることを確認できるはずです。  

ここで重要なことを追記致します。NSTimerを用いてBackgroundで位置情報を定期的に取得できるのは**標準位置情報サービス(startUpdatingLocation)**のときのみです。**大幅変更位置情報サービス(startMonitoringSignificantLocationChanges)**ではできません。これは忘れがちなので注意が必要です。  

因みに、Foreground, Backgroundと聞き慣れないかもしれないので説明しておくと  
・Foreground起動：端末にアプリの起動画面が表示されている状態  
・Background起動：端末にアプリの画面は表示されていないが、ホームボタン２回押しでアプリのアイコン(iOS6以前)またはアプリの画面(iOS7以降)が確認できる状態  
ということです。  
![アプリのBackground起動](/images/ios_bg_app.png)  

####最後に
以上、少しは参考になったでしょうか。
そのうち本日、紹介した機能を使って実際にアプリを開発してみようと思います。では今日はこの辺で。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- 広告2 -->
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
