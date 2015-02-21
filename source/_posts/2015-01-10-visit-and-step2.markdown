---
layout: post
title: "CLVisitとCMPedometerをフィールド検証してみた！"
date: 2015-01-10 22:30
comments: true
categories: ios ios8 CLVisit CMPedometer
---

####CLVisitとCMPedometerのフィールド検証
2015年初更新です。  
2014年の年末から2015年の年始にかけて海外に行く機会があったため、
以前、本ブログにも掲載したCLVisitとCMPedometerについてフィールド検証をしてみました。  
[iOS8.1でCLVisitとCMPedometerを試してみた！](http://grandbig.github.io/blog/2014/11/30/visit-and-step/)を参照ください。  
(思いつきで、出発前夜にアプリを急造しました)  

CLVisitは大幅変更位置情報サービスと同様に極力バッテリを消費しない仕組みで作られていることと思います。  
これが『旅行の思い出を残したいというユーザの想いとマッチするのでは？』と考えたこともあり、現実的にどこまで使えるものなのか検証してみました。  

<!-- more -->

#####アプリの機能
検証に利用したアプリの機能は下記です。

* CLVisitで取得した結果を端末内部のDBに保存  
* CMPedometerで取得した結果を端末内部のDBに保存  
* CLVisitでデータを取得できたタイミングで、結果をローカルプッシュ
* CMPedometerでデータを取得したタイミングで、結果をローカルプッシュ

#####ソースコード
ソースは[GitHub](https://github.com/grandbig/VisitTheWorld)にアップしています。  

CLVisitとCMPedometerの部分のみピックアップすると下記のように実装しています。  
まずはCLVisitの処理は下記です。

```objective-c
// ViewController.m
// CLVisitの処理
- (void)locationManager:(CLLocationManager *)manager didVisit:(CLVisit *)visit
{
	// 値の取得
	float lat = visit.coordinate.latitude;                              // 緯度
	float lng = visit.coordinate.longitude;                             // 経度
	float hacc = visit.horizontalAccuracy;                              // 水平精度
	NSDate *arrivalDate = visit.arrivalDate;                            // 到着日時(NSDate型)
	NSDate *departureDate = visit.departureDate;                        // 出発日時(NSDate型)
	double arrivalUnixTime = [arrivalDate timeIntervalSince1970];       // 到着日時(timestamp)
	double departureUnixTime = [departureDate timeIntervalSince1970];   // 出発日時(timestamp)
	
	// NSDate型の日時をNSString型に変換
	NSDateFormatter *outputFormatter = [NSDateFormatter alloc] init];
	[outputFormatter setDateFormat:@"yyyy-MM-dd HH:mm:ss"];
			    
	NSString* arrivalDateString = [outputFormatter stringFromDate:arrivalDate];
	NSString* departureDateString = [[outputFormatter stringFromDate:departureDate];

	// データの挿入
	[self insertPlaceData:lat longitude:lng horizontalAccuracy:hacc arrivalDate:arrivalDateString departureDate:departureDateString arrivalDateUT:arrivalUnixTime departureDateUT:departureUnixTime];
	// ローカルプッシュ
	NSString *message = [[NSString alloc] initWithFormat:@"lat: %f, lng: %f, hacc: %f, 到着日時: %@, 出発日時: %@", lat, lng, hacc, arrivalDateString, departureDateString];
	[self sendLocalNotificationForMessage:message soundFlag:NO];
}
```

次にCMPedometerの処理を下記です。  

```objective-c
// VisitController.m
// CMPedometerの処理
/**
 歩数の計測開始処理
*/
- (void)startPedometer
{
	self.pmp = [CMPedometer alloc] init];]
	[self.pmp startPedometerUpdatesFromDate:[NSDate date]
								withHandler:^(CMPedometerData *pedometerData, NSError *error) {
									dispatch_async(dispatch_get_main_queue(), ^{
										// step数の取得
										NSNumber *step = pedometerData.numberOfSteps;
										NSInteger stepInt = [step integerValue];

										// 距離
										NSNumber *distance = pedometerData.distance;
										NSInteger distanceInt = [distance integerValue];

										// 開始日時, 終了日時のUnixTimeを取得
										double startDateUnixTime = [pedometerData.startDate timeIntervalSince1970];
										double endDateUnixTime = [pedometerData.endDate timeIntervalSince1970];

										// 日時取得のためにFormatを設定
										NSDateFormatter *outputFormatter = [[NSDateFormatter alloc] init];
										[outputFormatter setDateFormat:@"yyyy-MM-dd HH:mm:ss"];
										// 開始日時, 終了日時のNSString型を取得
										NSString* startDateString = [outputFormatter stringFromDate:pedometerData.startDate];
										NSString *endDateString = [outputFormatter stringFromDate:pedometerData.endDate];

										// 階段の昇降
										NSNumber *floorsAscended = pedometerData.floorsAscended;
										NSNumber *floorsDescended = pedometerData.floorsDescended;
										NSInteger floorsAscendedInt = [floorsAscended integerValue];
										NSInteger floorsDescendedInt = [floorsDescended integerValue];

										// データの挿入
										[self insertMotionData:stepInt distance:distanceInt startDate:startDateString endDate:endDateString startDateUT:startDateUnixTime endDateUT:endDateUnixTime floorsAscended:floorsAscendedInt floorsDescended:floorsDescendedInt];

										// ローカルプッシュ通知
										NSString *message = [NSString alloc] initWithFormat:@"Step Count Result\n\n歩数: %@\n距離: %@[m]\n開始時間: %@\n終了時間: %@\n上った回数: %@\n下りた回数: %@", step, distance, startDateString, endDateString, floorsAscended, floorsDescended];
										[self sendLocalNotificationForMessage:message soundFlag:NO];]
									});
								}];
}
```

#####結果
では、結果を見てみましょう。  
まずは、CLVisitの結果です。  

![CLVisitの結果](/images/visit_test1.png)  
  
* 取得期間: 2014/12/28 14:30 〜 2015/01/03 20:00  
* 取得数: 117点 (オランダ: 66点, ベルギー: 49点, ルクセンブルク: 2点)  
* 精度: 0〜50[m]: 34点, 50〜100[m]: 76点, 100[m]〜: 7点  
* 到着日時と出発日時が正しく取れた: 58点, 到着日時のみ正しく取れた: 0点, 出発日時のみ正しく取れた: 59点  
    * CLVisitでは到着日時が正しく取れない場合は **0001-01-01 01:00:00** (月日や日時が異なる値になる場合もある)となり、出発日時が正しく取れない場合は **4001-01-01 01:00:00**  (同じく、月日や日時が異なる値になる場合もある)となる。
* 10分程度の滞在時間でも取得できる
* 歩き続けている間は取得できない(歩行速度程度の遅い速度でも移動していると取得できない)
* 数 [m] 〜 数十 [m]の距離間でも取得できる

次に、CMPedometerの結果です。  

* 取得期間: 2014/12/28 16:02:07 〜 2015/01/03 12:07:45
* 取得数: 1120点
* ステップ数: 57998
* 移動距離: 38623 [m]
* 階段を上がった回数: 36, 階段を下りた回数: 49
* 計測を開始してから明示的に終了するまで取り続けることができなかった(2度 起動し直した)
* 歩行時と電車乗車時の区別はできていなかった

#####今後の展望
海外旅行の思い出として軌跡を残すことを考えたアプリは既にたくさんあります。  
主にiPhoneで写真を撮影したタイミングで緯度・経度を取得してプロットするというものが多いことと思います。  
CLVisitはこれに対する補助機能としての役割を多少期待できるかもしれません。  
とは言え、『滞在 = 写真を撮る』と考えると同じ地点にプロットが集まるのかもしれませんが...。  
CMPedometerは歩行時のみであることを保証できないため、iPhoneの移動が停止/歩行/走行/電車などを区別するためにCMMotionActivityを合わせて利用したいと思いました。  
それにより、万歩計としての精度を高めることができるかもしれません。  
(う〜ん。今回、実装しておけば良かった...)  

と言ったところで、本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
