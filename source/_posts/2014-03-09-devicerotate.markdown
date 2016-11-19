---
layout: post
title: "Universalアプリ開発における縦横対応について"
date: 2014-03-09 21:39
comments: true
categories: ios
---

###iPhoneは縦のみ対応/iPadは縦横対応させたい！！
さて、久々に基本的なところで躓いてしまったのでメモしておきます。iPhone/iPadの両方に対応したUniversalなアプリを開発するときに、縦横対応を考える必要があります。iPhoneであれば基本的には縦もしくは横どちらかに固定すれば十分です。一方でiPadは縦をメインとして使うユーザと横をメインとして使うユーザの両方がターゲットとなります。では、iPhone/iPadで固定/非固定を分けて開発するためにはどうしたら良いのでしょうか？  

今回はその答えを始めに紹介し、続いて筆者が躓いたことによって得た情報を紹介したいと思います。  

<!--more-->

では、始めにiPhone/iPadで固定/非固定を分けて開発する方法を紹介しましょう。  
Xcode5上で**TARGETS > Deployment Info**を見て下さい。iPhoneとiPadの文字が並んで表示されている部分があると思います。(恐らく、iPhoneが青字になっていることでしょう。)  
この下に**Device Orientation**と書かれており、**Portrait, Upside Down, Landscape Left, Landscape Right**の4つのチェックボックスがあります。  
iPhoneは縦のみ, iPadは全方向OKにしたい場合は、iPhone選択時に**Portraitのみにチェック**, iPad選択時に**4つ全てにチェック**を入れましょう。  
![Xcode5で各デバイスの設定をしましょう](/images/devicerotate1.png)  

これで設定完了です。非常に簡単ですね！  
筆者はソースコードでどうにかしようと取り組んでいたため、時間がかかってしまいました笑。  
ただ、逆に言えばソースコードでも設定は可能なんです。そこについて紹介しましょう。  

まず、筆者は下記のようにAppDelegate.mファイルにソースを書きました。  
```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
	self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
	self.window.backgroundColor = [UIColor whiteColor];

	// UIViewController型のViewControllerを生成
	ViewController *vc = [[ViewController alloc] init];
	vc = (ViewController *)[[NavigationController alloc] initWithRootViewController:vc];
	self.window.rootViewController = vc;
	[self.window makeKeyAndVisible];

	return YES;
}
```objective-c
NavigationControllerをrootViewControllerに指定しています。そのNavigationControllerに自身で作成したViewControllerを加えています。  
さて、続いてViewController.mのソースを書いていきます。Objective-Cでは端末が回転したときに**supportedInterfaceOrientations**と**shouldAutorotate**の2つが呼び出されます。  
supportedInterfaceOrientationsは回転を許可する端末の方向を指定し, shouldAutorotateは端末が回転したときに自動で回転するかどうかを指定します。  
例えば、  
```
- (NSInteger)supportedInterfaceOrientations
{
	return UIInterfaceOrientationMaskPortrait;
}

- (BOOL)shouldAutorotate
{
	if(self.interfaceorientation == UIInterfaceOrientationPortrait) {
		return YES;
	} else {
		return NO;
	}
}
```
と書くと、supportedInterfaceOrientationsではPortrait方向のみ回転を許可し、shouldAutorotateではPortrait方向のみ自動で回転するようになっています。この両関数の指定がずれた場合はエラーが発生します。  
上記のように書いておけば、冒頭で説明したXcode5のDevice Orientationの4つ全てにチェックを入れていたとしても、回転することはありません。  
のはずでしたが、まだこの状態では回転してしまいます...。なぜなんでしょうか？  
それはrootViewControllerにNavigationControllerを設定しまったためだそうです。これを回避するためには、NavigationControllerに設定してあるtopViewControllerの値を返すようにCustomNavigationControllerを作成する必要があります。  
では、UINavigationControllerを継承したCustomNavigationControllerを作成しましょう。  
```objective-c
// CustomNavigationController.h
#import <UIKit/UIKit.h>
@interface CustomNavigationController : UINavigationController

@end
```

```objective-c
#import "CustomNavigationController.h"

@interface CustomNavigationController ()

@end

@implementation CustomNavigationController

- (id)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil
{
	self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
	if(self) {
		// Custom initialization
	}

	return self;
}

- (void)viewDidLoad
{
	[supper viewDidLoad];
}

- (void)didReceiveMemoryWarning
{
	[super didReceiveMemoryWarning];
}

// 新たに加えるソースは下記4行のみです。
- (NSInteger)supportedInterfaceOrientations
{
	return self.topViewController.supportedInterfaceOrientations;
}

@end
```
ここまで書ければ、後はAppDelegate.mファイルでNavigationControllerとしていたところをCustomNavigationControllerに変更すればOKです。  
今度こそ、端末の回転を抑制することができていますよね？  

今回は初歩的な話でしたが(いつもか...笑)、基本は非常に重要であり、知っていないと思わぬところで時間をくってしまいます。  
様々な書籍でObjective-Cの基本的な書き方, Xcodeの基本的な使い方が紹介されていると思いますが、筆者のおすすめは下記。  
**WEB DB PRESS**で初めてiOS7について基本的なところを説明してくれています。初心者には役に立つこと間違いないでしょう。**Software Design Plus iOSアプリ エンジニア養成読本**はまだ未発売ですが、iOSアプリのエンジニアとして生き残るための方法が書いてあるとのことなので、今から楽しみにしています。といったところで本日はここまで。
<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?t=grandbig7-22&o=9&p=8&l=as1&asins=4774162876&ref=qf_sp_asin_til&fc1=000000&IS2=1&lt1=_blank&m=amazon&lc1=0000FF&bc1=000000&bg1=FFFFFF&f=ifr" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?t=grandbig7-22&o=9&p=8&l=as1&asins=4774163856&ref=qf_sp_asin_til&fc1=000000&IS2=1&lt1=_blank&m=amazon&lc1=0000FF&bc1=000000&bg1=FFFFFF&f=ifr" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>


