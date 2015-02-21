---
layout: post
title: "Mapに好きな画像を配置しよう！"
date: 2013-09-28 18:27
comments: true
categories: ios map
---

###Mapにピンではなく、自分の好きな画像を配置したい！！
さて、筆者は思った以上にハマってしまいました。マップを表示して、よく見る赤ピンを配置するまでは余裕だったのですが、如何せんその先が上手くいかず時間がかかってしまいました。ネット上でいろいろとヒントをもらいましたが、なかなかわかりづらかったのでブログにメモしておきます。  

<!--more-->

####Mapを表示しよう
iOS6以降から実装されたApple独自のMapを嫌う人もいるかもしれませんが、実装は最も簡単です。(iOS5以前であればGoogleのMapが表示されます。)  
まず、AnnotationPracticeという名前でプロジェクトを作成します。  
例によって[Let's Create an iPhone App](http://grandbig.github.io/blog/2013/09/13/lets-create-an-iphone-app/)のときと同様にViewControllerの追加まで進めます。そして、今回はMapを表示するので、**MapKit.framework**をライブラリに追加します。  
![AnnotationPractice.xcodeprojを作成](/images/annotationpractice1.png)  

AppDelegate.mファイルでViewController.xibに設定している画面を表示できるよう設定することを忘れずに。(リマインドとして下記に書いておきます。)
```
// AppDelegate.mファイルに下記を書くこと
#import "AppDelegate.h"
#import "ViewController.h"

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
	self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen] bounds];
	self.window.backgroundColor = [UIColor whiteColor];
	[self.window makeKeyAndVisible];
	UIViewController *viewCtl = [[ViewController alloc] init];
	self.window.rootViewController = viewCtl;
								    
	return YES;
}
```

さて、次にViewController.hファイルを下記のようにします。
```
// ViewController.hファイルの内容
#import <UIKit/UIKit.h>
#import <MapKit/MapKit.h>

@interface ViewController : UIViewController <MKMapViewDelegate> {
	    IBOutlet MKMapView *mapView;
}

@end
```

また、ViewController.xibでMapを貼り付けて、そのMapとViewController.hで定義したmapViewを結びつけます。  
![Map ViewをViewController.xibに貼り付け](/images/annotationpractice2.png)  

さあ、これで端末を繋いで実行すればマップが下のように表示されます。
![Mapが表示されます](/images/annotationpractice3.png)  

####赤いピンを配置しよう
画像を配置したいのですが、ここはぐっと我慢してまずはデフォルトの赤ピンを配置してみることにします。  

新たにCustomAnnotationクラスを作成します。  
＜手順＞  
①プロジェクトを右クリックして**New File...**  
②Subclass ofで**NSObject**を選択  
③ClassをCustomAnnotationとする(クラス名なので好きな名前でOK)  

追加できたら、早速CustomAnnotation.hとCustomAnnotation.mの両ファイルを編集しましょう。
```
// CustomAnnotation.hファイル
#import <Foundation/Foundation.h>
#import <MapKit/MapKit.h>

@interface CustomAnnotation : NSObject <MKAnnotation> {
	CLLocationCoordinate2D coordinate;
	IBOutlet NSString *annotationTitle;
	IBOutlet NSString *annotationSubtitle;
}

@property (nonatomic, readonly) CLLocationCoordinate2D coordinate;
- (id)initWithLocationCoordinate:(CLLocationCoordinate2D) _coordinate
                           title:(NSString *)_annotationTitle subtitle:(NSString *)_annotationannSubtitle;
- (NSString *)title;
- (NSString *)subtitle;

@end
```
CLLocationCoordinate2Dの型ではIBOutletを指定できないので、少々面倒ですが、CustomAnnotation.hに@propertyを書き、CustomAnnotation.mファイルに@synthesizeを書きます。
```
// CustomAnnotation.mファイル
#import "CustomAnnotation.h"

@implementation CustomAnnotation
@synthesize coordinate;

- (NSString *)title {
	    return annotationTitle;
}

- (NSString *)subtitle {
	    return annotationSubtitle;
}

- (id)initWithLocationCoordinate:(CLLocationCoordinate2D) _coordinate
                           title:(NSString *)_annotationTitle subtitle:(NSString *)_annotationSubtitle {
	coordinate = _coordinate;
	self->annotationTitle = _annotationTitle;
	self->annotationSubtitle = _annotationSubtitle;
	return self;
}

@end
```
initWithLocationCoordinate関数を定義して初期化時に位置(緯度・経度), 吹き出しのタイトル, 吹き出しのサブタイトルを設定したannotationを返すようにします。  

そしてViewController.mファイルを下記のように編集して、赤いピンを配置します。
```
#import "ViewController.h"
// ①CustomAnnotation.hファイルをimport
#import "CustomAnnotation.h"

@interface ViewController ()

@end

@implementation ViewController

- (id)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil
{
	self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
	if (self) {
		// Custom initialization
	}
	return self;
}

- (void)viewDidLoad
{
	[super viewDidLoad];

	// ②CustomAnnotationクラスの初期化(インスタンス化)
	CustomAnnotation *ann1 = [[CustomAnnotation alloc] initWithLocationCoordinate:CLLocationCoordinate2DMake(35.685623, 139.763153) title:@"TEST" subtitle:@"test"];
	// ③ついでにユーザの現在地を表示するように設定
	[mapView setShowsUserLocation:YES];
	// ④annotationをマップに追加
	[mapView addAnnotation:ann1];
}

- (void)didReceiveMemoryWarning
{
	[super didReceiveMemoryWarning];
}

@end
```

これでプロジェクトを実行してみましょう。以下のようになるはず。  
![Mapに赤いピンを配置](/images/annotationpractice4.png)  

####Mapに好きな画像を配置しよう
よし、これでやっと画像配置の段階に移れそうです。  
赤いピンを好きな画像に置き換えるためにはMKAnnotationViewクラスを使います。まずはViewController.mファイルのviewDidLoad関数内に1行追加します。

```
- (void)viewDidLoad
{
	[super viewDidLoad];

	CustomAnnotation *ann1 = [[CustomAnnotation alloc] initWithLocationCoordinate:CLLocationCoordinate2DMake(35.685623, 139.763153) title:@"大手町駅" subtitle:@"千代田線"];
	// 下記１行を追加
	[mapView setDelegate:self];
	[mapView setShowsUserLocation:YES];
	[mapView addAnnotation:ann1];
}

```
これを追加することでaddAnnotationを実行した後にviewForAnnotation関数内の処理を実行するようになります。viewForAnnotation関数で実行したい処理を書きます。(もちろんViewController.mファイルに記述します。)

```
// ViewController.mファイル内にviewForAnnotation関数を記述
-(MKAnnotationView*)mapView:(MKMapView*)_mapView viewForAnnotation:(id <MKAnnotation>)annotation {
	// ①ユーザの現在地はデフォルトの青丸マークを使いたいのでreturn: nil
	if (annotation == mapView.userLocation) {
		return nil;
	} else {
		MKAnnotationView *annotationView;
		// ②再利用可能なannotationがあるかどうかを判断するための識別子を定義
		NSString* identifier = @"Pin";
		// ③dequeueReusableAnnotationViewWithIdentifierで"Pin"という識別子の使いまわせるannotationがあるかチェック
		annotationView = (MKAnnotationView*)[mapView dequeueReusableAnnotationViewWithIdentifier:identifier];
		// ④使い回しができるannotationがない場合、annotationの初期化
		if(annotationView == nil) {
			annotationView = [[MKAnnotationView alloc] initWithAnnotation:annotation reuseIdentifier:identifier];
		}
		// ⑤好きな画像をannotationとして設定
		annotationView.image = [UIImage imageNamed:@"foot_30.png"];
		annotationView.annotation = annotation;
		return annotationView;
	}
}

```

これでプロジェクトを実行すると、めでたく自分で設定した画像が表示されます。  
![足あと画像がMapに表示](/images/annotationpractice5.png)  


####吹き出しをつける
画像をタップしたときに吹き出しを出すようにしましょう。  
viewForAnnotation関数内で設定を追加します。
```
annotationView.image = [UIImage imageNamed:@"foot_30.png"];
// ここに追加
annotationView.canShowCallout = YES;
annotationView.annotation = annotation;
```

今回の例ではtitleにTEST, subtitleにtestとしましたが、例えば何らかの取得した値を表示したいということもあるでしょう。その場合、数値, 日付などをNSString型に変換した上で設定する必要があります。ここで結構ハマりました(汗)  
何にハマったかというとNSStringの初期化です。  
```
// 失敗例(メモリリークが発生する例)
NSString *str1 = [NSString stringWithString:@"test"];
NSString *str2 = [NSString stringWithFormat:@"test:%f",123];

// 成功例
NSString *str1 = [NSString initWithString:@"test"];
NSString *str2 = [NSString initWithFormat:@"test:%f",123];
```
stringWithString, stringWithFormatを使用した場合、簡易コンストラクタで生成され、releaseタイミングがNSStringに委ねられるとのことです。なので、アプリ起動時に画像をタップするときにはメモリが解放されてしまっているためにメモリリークエラーを引き起こすものと思われます。  
initWithString, initWithFormatでコンストラクタを生成するようにした方が良さそうです。  
次のサイトを参考にしました。([iPhoneアプリ開発備忘録](http://iphone-app-developer.seesaa.net/article/183323144.html))


基本中の基本かもしれませんが、一つ一つ理解しながら開発できたので、良い勉強になりました。
