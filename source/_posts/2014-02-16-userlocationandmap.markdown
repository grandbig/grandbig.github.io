---
layout: post
title: "MKMapViewの初期中心点をユーザの現在地に設定する方法"
date: 2014-02-16 21:16
comments: true
categories: ios map
---

###MKMapViewの中心位置について考えた
本日はMKMapViewの初期表示について説明します。MKMapViewを扱うときにはユーザの位置情報を用いることが多いと思います。そのほとんどの場合、マップを初期表示したときにユーザの現在地を中心にしたいのではないかと思います。しかしながらiPhoneで位置情報を取得するとき、アプリを起動して即座に現在地を特定することはできません。では、『どうするのか？』について失敗談を交えつつ紹介させて頂きます。  


<!--more-->

まずは失敗例から紹介します。  
筆者は**ユーザの現在地をマップの中心位置に設定する**ということに意識が向きすぎたせいか当初は下記の手段を取っていました。  

下記はViewController.h  
```
#import <UIKit/UIKit.h>
#import <CoreLocation/CoreLocation.h>
#import <MapKit/MapKit.h>

@interface ViewController : UIViewController <MKMapViewDelegate, CLLocationManagerDelegate>

@property (strong, nonatomic) IBOutlet CLLocationManager *locationManager;
@property (weak, nonatomic) IBOutlet MKMapView *map;
```

下記はViewController.m  
```
// ユーザの位置を取得
CLLocation *location = [locationManager location];
CLLocationCoordinate2D uCoordinate = [location coordinate];
// マップの中心位置を設定
MKCoordinateRegion region = MKCoordinateRegionMake(uCoordinate, MKCoordinateSpanMake(0.05, 0.05));
```
しかし、これではアプリ起動直後に真っ青な画面が表示されてしまうことでしょう。その理由はiPhone側で位置情報を取得する前にマップを表示しようとしてしまっているためです。結果、ユーザの現在地はlatitude, longitude共に0として返ってきます。  
ここで筆者はマップに現在地のピンを表示することは割りと早くできていることに気づきました。であれば何かマップからユーザ情報が取得できるのではないかと。  
で考えたのが下記。(ViewController.mのみ修正)  
```
// ユーザの位置を取得
CLLocation *userLocation = self.map.userLocation.location;
CLLocationCoordinate2D uCoordinate = CLLocationCoordinate2DMake(userLocation.coordinate.latitude, userLocation.coordinate.longitude);
// マップの中心位置を設定
MKCoordinateRegion region = MKCoordinateRegionMake(uCoordinate, MKCoordinateSpanMake(0.05, 0.05));
```

しかし、これまたアプリ起動後に真っ青な画面が表示されてしまいました...。う〜む、なかなかうまくいかない。  

で、結局、下記で解決。(ViewController.mのみ修正)  
```
// マップにユーザの現在地を表示
self.map.showsUserLocation = YES;
// マップの中心地がユーザの現在地を追従するように設定
[self.map setUserTrackingMode:MKUserTrackingModeFollow];
```
位置情報が取れ次第(ユーザの現在地が特定でき次第)、マップの中心地が移動するのでアプリ起動後に割りとすぐ理想の結果が得られます。(因みにマップを動かせば、追従モードはOFFとなります。)


