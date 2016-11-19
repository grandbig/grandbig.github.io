---
layout: post
title: "端末のバッテリ残量を取得(iOS8)"
date: 2014-11-08 16:45
comments: true
categories: ios ios8
---

####端末のバッテリ残量の取得が細かくなった
本日はiPhoneのバッテリ残量の取得について書きます。  
これまではソース上、5%刻みでしかバッテリ残量を取得できませんでした。  
しかし、iOS8以上の端末であれば、 **1%刻み** でのバッテリ残量の取得が可能になったようです。  

<!-- more -->

バッテリの取得手法は下記のまま変わりません。  

```objective-c
// ViewController.m

#import "ViewController.h"

@interface ViewController ()

@property(strong, nonatomic) UIDevice *device;

@end

@implementation ViewController

- (void)viewDidLoad
{
	[super viewDidLoad];

	// デバイス情報の設定
	self.device = [UIDevice currentDevice];
	self.device.batteryMonitoringEnabled = YES;
}

<省略>

// バッテリ残量をログ出力
- (void)outputDeviceBattery
{
	float level = self.device.batteryLevel;
	NSLog(@"level: %f", state, level);
}

// ボタンタップでバッテリ残量ログを出力
- (IBAction)btnAction:(id)sender {
	[self outputDeviceBattery];
}

```

ここまで実装できたら、iOS7.xとiOS8.x端末で計測してみてください。  
iOS7.xであれば、『0.450000』のように5%刻みですが、  
iOS8.xであれば、『0.310000』のように1%刻みで値が返ってくることがわかります。  
これで少しは正確にバッテリ情報を取得できるようになりましたね。  

今日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

