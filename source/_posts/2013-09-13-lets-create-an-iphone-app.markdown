---
layout: post
title: "Let's create an iPhone App"
date: 2013-09-13 23:35
comments: true
categories: ios
---

###Flash Lightアプリを作ろう！
最近iPhoneに実装されたLEDライトの点滅間隔を調べる機会がありました。  
そこで、そのときに作った、簡単な『Flash Light アプリ』を紹介しましょう！  

<!--more-->

####ViewControllerを新規追加
まずは表示に必要なViewControllerを追加しましょう！  
1: プロジェクトを右クリックして、『New File...』を選択  
2: Objective-c Classを選択してNext  
3: Class 『ViewController』, Subclass of 『UIViewController』を選択してNext  
4: xcodeprojファイルと同じ場所に保存  

####AppDelegateファイルを編集
次にViewControllerをアプリ起動時に表示できるように設定しましょう。  
そのためにはAppDelegateファイルを編集する必要があります。では『AppDelegateファイル』とは何でしょうか？  
AppDelegateファイルとは簡単に言えば、アプリの起動/停止をトリガーとして何かアクションを起こす役目を持っています。  
これはアプリを開発する上で非常に重要なファイルと言えるでしょう。  

AppDelegate.mファイル内でViewControllerを扱うために
```
#import "AppDelegate.h"
#import "ViewController.h"

@implementation AppDelegate
```
とViewControllerをimportしましょう。  

では実際にViewControllerをアプリwindowに設定します。
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
	// ウィンドウの初期化と生成
	_window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
	// 背景色を指定(今回は黒色)
	_window.backgroundColor = [UIColor blackColor];
	// ウィンドウを表示
	[_window makeKeyAndVisible];

	// ViewControllerの初期化と生成
	UIViewController* viewCtl = [[[ViewController alloc] init] autorelease];
	// ViewControllerをウィンドウに設定
	_window.rootViewController=viewCtl;
	return YES;
}
```

####ViewController.h/ ViewController.mを編集
さて、これでViewControllerに実装したものをウィンドウに表示する設定が終わりました。  
今度はLEDライトの点滅方法を考えるわけですが、そもそもLEDライトってどうやって扱うんでしょうか？  

答えは『AVFoundation.framework』を使います！  
因みにライブラリ追加は以下の手順。  
1: プロジェクトを選択  
2: TARGETSを選択  
3: BuildPhasesを選択  
4: Link Binary With Librariesを選択  
5: +を選択して『AVFoundation.framework』を追加  
![AVFoundation.frameworkを追加](/images/add_framework.png)

ただ、安心してはいけません。必ず使うものはimportしましょう！  
ViewController.hファイルにimport
```
#import <UIKit/UIKit.h>
// 以下のAVFoundationを追加
#import <AVFoundation/AVFoundation.h>

@interface ViewController : UIViewController {
	...
```

いきなり自動点滅は難しいので、まずはスイッチでLEDのON/OFFを切り替えられるようにしてみましょう。  
ViewController.hを引き続き編集
```
@interface ViewController : UIViewController {
	// スイッチを定義
	IBOutlet UISwitch *sw;
}

// スイッチのON/OFF切り替え用関数を宣言
-(IBAction)switchChanged:(id)sender;
```
次にViewController.mを編集します。
```
// スイッチのON/OFF切り替え用関数の処理を定義
- (IBAction)switchChanged:(id)sender {
	// 背面カメラを初期化
	AVCaptureDevice *captureDevice = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];

	// スイッチがONのとき
	if(sw.on) {
		// LEDの点灯は下記3行
		[captureDevice lockForConfiguration:NULL];
		captureDevice.torchMode = AVCaptureTorchModeOn;
		[captureDevice unlockForConfiguration];
	// スイッチがOFFのとき
	} else {
		// LEDの消灯は下記3行
		[captureDevice lockForConfiguration:NULL];
		captureDevice.torchMode = AVCaptureTorchModeOff;
		[captureDevice unlockForConfiguration];
	}
}
```

####ViewController.xibファイルを編集
そしてViewController.xibを編集します。  
xibの編集はアプリに表示する画面を直接編集することに相当します。  


1: 右下リストからSwitchを選択し、ドラッグ&ドロップでViewにスイッチを持ってきます。  
2: 定義したswとViewに載せたSwitchを結びつけるために、左側メニューにあるPlaceholdersのFile's Ownerを右クリック  
3: Outletsのswの右端◯からドラッグ&ドロップでView上のスイッチまで持っていきます。  
4: 同様にswitchChangedも右端◯からドラッグ&ドロップでView上のスイッチに持っていき、Value Changedを選択  

![スイッチの結びつけ](/images/edit_xib.png)

以上で手動でのLEDのON/OFFができました。動作は実機で確認しましょう！(シミュレータはLEDないので…)  

####LEDのON/OFFを自動化
LEDのON/OFFを自動化するためにNSTimerを使いましょう！  
ViewController.hにNSTimerを宣言  
```
@interface ViewController : UIViewController {
	...
	IBOutlet NSTimer *timer;
	...
}
```
ViewController.mを編集
```
// LEDの現在のON/OFF状態を判別するためにフラグを設定
static BOOL ledFlag = NO;
...
- (void)viewDidLoad {
	[super viewDidLoad];
	
	// timerを設定(1秒ごとに処理を実行)
	timer = [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(ledChange:)userInfo:nil repeats:YES];
}

// timerで設定した定期間隔ごとに処理される内容を定義
- (void)ledChange:(NSTimer *)theTimer {
	AVCaptureDevice *captureDevice = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];

	// LEDがOFFだった場合はONにする
	if(ledFlag == NO) {
		[captureDevice lockForConfiguration:NULL];
		captureDevice.torchMode = AVCaptureTorchModeOn;
		[captureDevice unlockForConfiguration];
		ledFlag = YES;
	// LEDがONだった場合はOFFにする
	} else {
		[captureDevice lockForConfiguration:NULL];
		captureDevice.torchMode = AVCaptureTorchModeOff;
		[captureDevice unlockForConfiguration];
		ledFlag = NO;
	}
```

これでLEDを1秒ごとに点滅させることができるようになりました。  

さ〜て、いかがだったでしょうか？  
今や簡単に誰でもアプリが作れる時代です。ぜひ、皆さんも作ってみて下さい。  
余談ですが、iOS7から懐中電灯機能はアプリをインストールするまでもなく搭載されることとなったので、懐中電灯アプリはAppleに申請しても通らないと思います(汗)
