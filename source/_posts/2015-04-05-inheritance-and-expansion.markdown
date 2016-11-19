---
layout: post
title: "Objective-Cにおける継承とカテゴリ"
date: 2015-04-05 01:10
comments: true
categories: ios
---

####Objective-Cの継承とカテゴリについて
さて、本日は今更ですが、Objective-Cの継承とカテゴリについて書きたいと思います。  
理由は、最近、自分だけではなく、 **『第三者が見てもわかりやすいクラス』** を作ることを意識し始めたからです。  
本当に今更ですね笑。  

アジェンダは下記です。  

* Objective-Cの継承
* Objective-Cのカテゴリ
* シングルトンパターンでの継承
* シングルトンパターンのクラスを継承したクラスのカテゴリ

では、早速見て行きましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####Objective-Cの継承
まずは、継承です。  
初めに元となるクラスを作成します。  

```objective-c
// SampleClass.h
#import <Foundation/Foundation.h>

@interface SampleClass : NSObject

@property(strong, nonatomic) NSString *sampleProperty;

- (void)sampleMethod;

@end
```

```objective-c
// SampleClass.m
#import "SampleClass.h"

@implementation SampleClass

- (id)init
{
	self = [super init];

	if(self) {
		self.sampleProperty = @"これはsamplePropertyです";
	}

	return self;
}

- (void)sampleMethod
{
	NSLog(@"sampleMethodを実行しました");
}

@end
```

このSampleClassを継承したクラスを作成します。  

```objective-c
// SampleClass2.h
#import "SampleClass.h"

@interface SampleClass2 : SampleClass

@property(strong, nonatomic) NSString *sample2Property;

- (void)sample2Method;

@end
```

```objective-c
// SampleClass2.m
#import "SampleClass2.h"

@implementation SampleClass2

- (void)sample2Method
{
	[self sampleMethod];
}

@end
```

このようにSampleClass2はSampleClassを継承しているので、sampleMethodを利用することができます。  
このクラスをViewControllerで使う場合は下記のようにできます。  

```objective-c
// ViewController.m
#import "SampleClass2.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];
	[self testMethod];
}

- (void)testMethod {
	// SampleClass2の初期化
	SampleClass2 *sample2 = [[SampleClass2 alloc] init];
	// SampleClass2で定義されたsample2Methodを実行
	[sample2 sample2Method];
	// スーパークラスであるSampleClassのsampleMethodを実行
	[sample2 sampleMethod];
}

@end
```

元々あったクラスに新たなプロパティやメソッドを追加したクラスを作成した場合はこのようにやれば良いわけです。  

####Objective-Cのカテゴリ
続いて、カテゴリです。  

SampleClassのカテゴリクラスを作成します。  

```objective-c
// SampleClass3.h
#import "SampleClass.h"

@interface SampleClass(SampleClass3)

- (void)sample3Method;

@end
```

```objective-c
// SampleClass3.m
#import "SampleClass3.h"

@implementation SampleClass(SampleClass3)

- (void)sample3Method
{
	return [self sampleMethod];
}

@end
```

上記を見ると、継承と何が違うの？と思われるかもしれません。  
実はカテゴリでは継承のようにプロパティの定義ができません。  
基本的に元のクラスに新しいメソッドを追加したいときに使うもののようです。  
[Wikipedia](http://ja.wikipedia.org/wiki/Objective-C)にも書いてある通り、『ひとつのクラスの実装を関連するメソッド群毎に別々の場所に分割して記述することを可能とする目的で作られた』だそうです。  

しかしながら、厳密にはプロパティも追加できます。  
追加したい場合は下記のようにします。  

```objective-c
// SampleClass3.h
#import "SampleClass.h"

@interface SampleClass(SampleClass3)

// ここを追加
@property(strong, nonatomic) NSString *sample3Property;

- (void)sample3Method;

@end
```

```objective-c
// SampleClass3.m
#import "SampleClass3.h"
// ここを追加
#import <objc/runtime.h>

@implementation SampleClass(SampleClass3)

// ここを追加
@dynamic sample3Property;

// ここを追加
- (void)setValue:(NSString*)sample3Property
{
	objc_setAssociatedObject(self, _cmd, sample3Property, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (void)sample3Method
{
	return [self sampleMethod];
}

@end
```

では、このカテゴリクラスをViewControllerで使ってみましょう。  

```objective-c
// ViewController.m
#import "SampleClass.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];
	[self testMethod];
}

- (void)testMethod {
	// SampleClassの初期化
	SampleClass *sample = [[SampleClass alloc] init];
	// SampleClassで定義されたsampleMethodを実行
	[sample sampleMethod];
	// SampleClass3のsample3Methodを実行
	[sample sample3Method];
}

@end
```

カテゴリは **元のクラスに定義を追加した** ものなので、上記ソースのように初期化するオブジェクトはあくまでも **SampleClass** であることに注意しましょう。  

####シングルトンパターンでの継承
シングルトンパターンでは、少し注意が必要です。  
元のクラスをシングルトンパターンにしましょう。  

```objective-c
// SampleClass.h
#import <Foundation/Foundation.h>

@interface SampleClass : NSObject

@property(strong, nonatomic) NSString *sampleProperty;

// シングルトンのインスタンス取得
+ (SampleClass *)sharedInstance;

- (void)sampleMethod;

@end
```

```objective-c
// SampleClass.m
#import "SampleClass.h"

@implementation SampleClass

// シングルトンのインスタンス取得
+ (SampleClass *)sharedInstance
{
	static SampleClass* sharedInstance = nil;
	static dispatch_once_t onceToken;
	dispatch_once(&onceToken, ^{
		sharedInstance = [[SampleClass alloc] init];
	});

	return sharedInstance;
}

- (id)init
{
	self = [super init];

	if(self) {
		self.sampleProperty = @"これはsamplePropertyです";
	}

	return self;
}

- (void)sampleMethod
{
	NSLog(@"sampleMethodを実行しました");
}

@end
```

SampleClassをシングルトンパターンにした上で、継承したSample2ClassをViewControllerで使ってみましょう。  

```
// ViewController.m
#import "SampleClass2.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];
	[self testMethod];
}

- (void)testMethod {
	// SampleClassで定義されたsampleMethodを実行
	[[SampleClass2 sharedInstance] sampleMethod];
	// SampleClass2で定義されたsample2Methodを実行
	[[SampleClass2 sharedInstance] sample2Method];
}

@end
```

実はこれだとうまくいきません。  
どこがうまくいかないかというと、 **SampleClass2で定義されたsample2Methodの実行** の部分です。  
これは **SampleClassにsample2Methodは定義されていません** といったエラーが表示されます。  
そう、初期化されたのはあくまでもSampleClassであって、SampleClass2ではないため、上記のようなエラーが出るのです。  

このエラーを防ぐためには、SampleClass2を下記のように定義します。  

```objective-c
// SampleClass2.h
#import "SampleClass.h"

@interface SampleClass2 : SampleClass

@property(strong, nonatomic) NSString *sample2Property;

+(SampleClass2 *)sharedInstance;

- (void)sample2Method;

@end
```

```objective-c
// SampleClass2.m
#import "SampleClass2.h"

@implementation SampleClass2

// シングルトンのインスタンス取得
+ (SampleClass2 *)sharedInstance
{
	static SampleClass2* sharedInstance = nil;
	static dispatch_once_t onceToken;
	dispatch_once(&onceToken, ^{
		sharedInstance = [[SampleClass2 alloc] init];
	});

	return sharedInstance;
}

- (void)sample2Method
{
	[self sampleMethod];
}

@end
```

はい。当たり前ですね。  
正しく理解していないと悩むことになるのでご注意を！

####シングルトンパターンのクラスを継承したクラスのカテゴリ
最後にややこしいことをしてみましょう笑。  
先ほどのシングルトンパターンのクラスであるSampleClassを継承したSampleClass2のカテゴリであるSampleClass4を作成してみます。  

```objective-c
// SampleClass4.h
#import "SampleClass2.h"

@interface SampleClass2(SampleClass4)

- (void)sample4Method;

@end
```

```
#import "SampleClass4.h"

@implementation SampleClass2(SampleClass4)

- (void)sample4Method
{
	[SampleClass2 sharedInstance].sample2Property = @"SampleClass4オブジェクトのsample2Propertyプロパティ";
}
```

さて、ViewControllerで使ってみましょう。  

```objective-c
// ViewController.m
#import "SampleClass4.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];
	[self testMethod];
}

- (void)testMethod {
	// SampleClassで定義されたsampleMethodを実行
	[[SampleClass2 sharedInstance] sampleMethod];
	// SampleClass2で定義されたsample2Methodを実行
	[[SampleClass2 sharedInstance] sample2Method];
	// SampleClass4で定義されたsample4Methodを実行
	[[SampleClass2 sharedInstance] sample4Method];
}

@end
```

なかなかに頭を混乱させられましたが、なんとかできました笑。  
といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
