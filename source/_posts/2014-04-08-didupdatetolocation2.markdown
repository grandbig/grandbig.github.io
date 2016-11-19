---
layout: post
title: "didUpdateLocationsを軽く調べてみる"
date: 2014-04-08 23:03
comments: true
categories: ios location
---

###didUpdateLocationsのNSArray型の引数が気になる！！
さて、今日はざっくり調べたことを書きます。ほんの数行です！  

<!--more-->

[前回](http://grandbig.github.io/blog/2014/04/05/didupdatetolocation/)、書きましたdidUpdateLocationsを使ってみました。  
最も気になったのが、NSArray型の引数であるlocationsの中身には何が入っているのか？ということです。  
下記のようにソースを書きます。  

```objective-c
- (void)locationManager:(CLLocationManager *)manager
	 didUpdateLocations:(NSArray *)locations
{
	CLLocation *currentLocation = locations.lastObject;
	NSInteger cnt = [locations count];
	CLLocationDegrees lat = currentLocation.coordinate.latitude;
	CLLocationDegrees lng = currentLocation.coordinate.longitude;

	NSLog(@"cnt: %ld,lat: %f, lng: %f", (long)cnt, lat, lng);
}
```
実際にログを出力してみた結果はというと...  
2014-04-08 23:02:35.371 LocationPractice[11243:60b] cnt: 1,lat: 35.933374, lng: 139.619913  
2014-04-08 23:02:36.837 LocationPractice[11243:60b] cnt: 1,lat: 35.933351, lng: 139.619884  
2014-04-08 23:02:37.428 LocationPractice[11243:60b] cnt: 1,lat: 35.933327, lng: 139.619856  
......

の繰り返しでした。  
NSArray型のlocationsには最新の位置情報しか入らないようです。  
う〜む。なんでNSArray型なんでしょうか...。ただ、よくよく考えるとNSMutableArray型でないから要素増えないんでしょうか...。何て思ったりしました。  
はい。短いけど、今日はここまで。
