---
layout: post
title: "一定時間内は画面タップをさせない！"
date: 2015-05-09 23:37
comments: true
categories: ios
---

####SVProgressHUDとNSTimerを使う
さて、本日はさくっとメモしておきます。  
できると思っていて当然できた話なので、本当にメモです笑。  

[AFNetworking](https://github.com/AFNetworking/AFNetworking)などでどこかしらのサーバに通信している間、画面をユーザに触らせたくないことがあるかもしれません。  
そんなときはローディングウィンドウを[SVProgressHUD](https://github.com/TransitApp/SVProgressHUD)で表示し、 **NSTimer** を使って時間で操ります。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

```objective-c

#import "ViewController.h"
#import "SVProgressHUD.h"

@interface ViewController ()

@property (strong, nonatomic) NSTimer *timer;

@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];
}

- (void)didReceiveMemoryWarning {
	[super didReceiveMemoryWarning];
}

- (IBAction)showLoadWindow:(id)sender {
	// ローディングウィンドウの表示
	[SVProgressHUD showWithStatus:@"Loading..." maskType:SVProgressHUDMaskTypeGradient];
	// タイマーの設定
	_timer = [NSTimer scheduledTimerWithTimeInterval:5.0 target:self selector:@selector(hiddenWindow:) userInfo:nil repeats:NO];
}

/**
	タイマー対象メソッド(ローディングウィンドウを非表示にする処理)
	@param timer タイマー
 */
- (void)hiddenWindow:(NSTimer *)timer
{
	[SVProgressHUD dismiss];
}

@end

```

よし、できた！ってところでメモ書き終了。

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
