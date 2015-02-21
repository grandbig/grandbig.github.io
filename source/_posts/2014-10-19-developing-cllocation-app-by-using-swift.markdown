---
layout: post
title: "Swiftを使って、位置情報を取得してみよう！"
date: 2014-10-19 22:41
comments: true
categories: ios location Swift
---

###Swiftを使って位置情報を使ってみよう
今日はSwiftを使った位置情報の取得を試してみたいと思います。  
今日もObjective-Cの書式と比較しながら進めてみます。  

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

<!--more-->

####インスタンス変数の宣言

* Objective-Cの場合

```objective-c
// ViewController.mファイル
@interface ViewController ()

@property (strong, nonatomic) CLLocationManager *lm;

@end
```
* Swiftの場合

```
// ViewController.swiftファイル
class ViewController: UIViewController {

	var lm:CLLocationManager?

	.....
}

```

####Delegate Protocolの宣言

* Objective-Cの場合

```objective-c
// ViewController.mファイル
@interface ViewController() <CLLocationManagerDelegate> {
	......
}
```

* Swiftの場合

```
// ViewController.swiftファイル
class ViewController: UIViewController, CLLocationManagerDelegate {
	......
}
```

####viewDidLoadメソッド

* Objective-Cの場合

```objective-c
// ViewController.mファイル
- (void)viewDidLoad {
	[super viewDidLoad];

	if([self.lm respondToSelector:@selector(requestAlwaysAuthorization)]) {
		[self.lm requestAlwaysAuthorization];
	}
}
```

* Swiftの場合

```
override func viewDidLoad() {
	super.viewDidLoad()

	if((self.lm?.respondToSelector("requestAlwaysAuthorization")) != nil) {
		self.lm?.requestAlwaysAuthorization()
	}
}
```

####CLLocationManagerDelegateメソッド

* Objective-Cの場合

```objective-c
// ViewController.mファイル
// 位置情報の許可設定
- (void)locationManager:(CLLocationManager *)manager didChangeAuthorizationStatus:(CLAuthorizationStatus)status
{
	if (status == kCLAuthorizationStatusNotDetermined) {
	} else if(status == kCLAuthorizationStatusAuthorizedAlways) {
		[self.lm startUpdatingLocation];
	} else if(status == kCLAuthorizationStatusAuthorizedWhenInUse) {
		[self.lm startUpdatingLocation];
	}
}

// 位置情報の更新ハンドリング処理
- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations {
	CLLocation *currentLocation = locations.lastObject;
	CLLocationDegrees lat = currentLocation.coordinate.latitude;
	CLLocationDegrees lng = currentLocation.coordinate.longitude;
	CLLocationDegrees alt = currentLocation.altitude;
	CLLocationDirection course = currentLocation.course;
	CLLocationSpeed spd = currentLocation.speed;
	CLLocationAccuracy hacc = currentLocation.horizontalAccuracy;
	CLLocationAccuracy vacc = currentLocation.verticalAccuracy;
	NSDate *dt = currentLocation.timestamp;
}
```

* Swiftの場合

```
// ViewController.swiftファイル
// 位置情報の許可設定
func locationManager(manager: CLLocationManager!, didChangeAuthorizationStatus status: CLAuthorizationStatus) {
	if(status == .NotDetermined) {
	} else if(status == .Authorized) {
		self.lm?.startUpdatingLocation()
	} else if(status == .AuthorizedWhenInUse) {
		self.lm?.startUpdatingLocation()
	}
}

// 位置情報の更新ハンドリング処理
func locationManager(manager: CLLocationManager!, didUpdateLocations locations: [AnyObject]!) {
	var currentLocation:CLLocation? = locations.last as? CLLocation
	var lat:CLLocationDegrees? = currentLocation?.coordinate.latitude
	var lng:CLLocationDegrees? = currentLocation?.coordinate.longitude
	var alt:CLLocationDegrees? = currentLocation?.altitude
	var course:CLLocationDirection? = currentLocation?.course
	var spd:CLLocationSpeed? = currentLocation?.speed
	var hacc:CLLocationAccuracy? = currentLocation?.horizontalAccuracy
	var vacc:CLLocationAccuracy? = currentLocation?.verticalAccuracy
	var dt:NSDate! = currentLocation?.timestamp
}
```

これでコーディングは完了です。  
Swiftでも位置情報が取得できるようになりました。  
次回はSwiftで何を書こうかな。  
といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
