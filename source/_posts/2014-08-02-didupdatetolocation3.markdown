---
layout: post
title: "iOS7とNSTimerと位置情報の取得について"
date: 2014-08-02 00:19
comments: true
categories: ios location
---

###iOS7で定期間隔での位置情報取得
本日は一言メモ程度に書きます。  
[Backgroundで位置情報を取得しよう！](http://grandbig.github.io/blog/2013/09/27/location-nstimer/)で紹介した**NSTimerで定期時間ごとに位置情報を取得してみよう！**の部分についてiOS7での注意しなければいけないことがありました。  

Background起動で位置情報の取得を可能な設定にしておくと、NSTimerが永続的に動くことについては以前お伝えしました。  
このNSTimerで設定した定期間隔ごとに下記のように位置情報の取得ができました。  

```objective-c
CLLocation *location = [locationManager location];
CLLocationCoordinate2D coordinate = [location coordinate];// 座標を取得
NSString *lat = [NSString alloc] initWithFormat:@"%f", coordinate.latitude];  // 経度を取得
NSString *lng = [NSString alloc] initWithFormat:@"%f", coordinate.longitude]; // 緯度を取得
NSLog(@"+++++ [デリゲートによらない場合]緯度,经度: %@, %@", lat, lng);
```

しかし、iOS7ではある記述をしないとうまくいかないことがわかりました。  

<!--more-->

それは**didUpdateLocationsメソッド**です。  
つまり、  

```
- (void)locationManager:(CLLocationManager *)manager
	 didUpdateLocations:(NSArray *)locations
{
	
}
```
の記述が必要なんです。  
これがなかったとしても、NSTimer自体は動くので、処理は定期的に実行されます。  
が、**位置情報の値が全く更新されません**。  
iPhoneを持って移動したとしても、１回目に取得した位置情報の値から変化がないのです。

もし、**NSTimerを使った定期間隔での位置情報の取得をiOS7で行う**のであれば、このことには十分に注意してください。  


<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
