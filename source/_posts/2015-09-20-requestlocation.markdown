---
layout: post
title: "iOS8でrequestLocationが使える！？"
date: 2015-09-20 01:21
comments: true
categories: ios ios8 location
---

####実はiOS8.4.1でrequestLocationが使える
さて、今回はたまたま気づいたバグ？の紹介です。  
皆様、ご存知の通り, iOS9から **requestLocation** なるものがCoreLocation.frameworkに追加されました。  
これにより、1回だけ現在地を取得する処理を自作せずにframeworkに任せることができるようになりました。  

しかし、iOS9からとアナウンスされているものの、実は **iOS8.4.1** でも利用できました。  
コードは下記になります。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

```objective-c
// ViewController.m
#import "ViewController.h"
#import <CoreLocation/CoreLocation.h>

@interface ViewController () <CLLocationManagerDelegate>

@property (strong, nonatomic) CLLocationManager *lm;

@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];

	self.lm = [[CLLocationManager alloc] init];
	self.lm.delegate = self;
	[self.lm requestWhenInUseAuthorization];
}

- (void)didReceiveMemoryWarning {
	[super didReceiveMemoryWarning];
}

- (void)locationManager:(CLLocationManager *)manager didChangeAuthorizationStatus:(CLAuthorizationStatus)status {
	if (status == kCLAuthorizationStatusNotDetermined) {
		// ユーザが位置情報の使用を許可していない
		NSLog(@"kCLAuthorizationStatusNotDetermined");
	} else if(status == kCLAuthorizationStatusAuthorizedAlways) {
		// ユーザが位置情報の使用を常に許可している場合
	 NSLog(@"kCLAuthorizationStatusAuthorizedAlways");
	} else if(status == kCLAuthorizationStatusAuthorizedWhenInUse) {
		// ユーザが位置情報の使用を使用中のみ許可している場合
		NSLog(@"kCLAuthorizationStatusAuthorizedWhenInUse");
	}
}

- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray<CLLocation *> *)locations {
	// 位置情報の取得に成功した場合
	NSLog(@"%@", locations.firstObject);
}

- (void)locationManager:(CLLocationManager *)manager didFailWithError:(NSError *)error {
	// 位置情報の取得に失敗した場合
	NSLog(@"%@", error);
}

- (IBAction)getLocation:(id)sender {
	if([self.lm respondsToSelector:@selector(requestLocation)]) {
		// iOS9.x以上の場合
		[self.lm requestLocation];
	} else {
		// iOS8.x以下の場合
		NSLog(@"iOS8.xです");
	}
}
```

騙されたと思って試してみてください。  
なぜかiOS8.4.1でも`requestLocation`が実行されますので。  


<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

