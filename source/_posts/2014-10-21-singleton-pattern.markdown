---
layout: post
title: "シングルトンパターンとクラスメソッドとインスタンスメソッド"
date: 2014-10-21 22:02
comments: true
categories: ios Singleton
---

####シングルトンを使用したクラスでクラスメソッドとインスタンスメソッドを定義する
本日は、シングルトンを使用したクラスの作成と、その中でクラスメソッドとインスタンスメソッドを定義した場合に、どう扱うのかについて見ていきます。  
早速、シングルトンパターンのクラスを定義しましょう。  
NSObjectを親クラスとしたSingletonSampleクラスを作成します。  

<!--more-->

```objective-c
// SingletonSample.hファイル
#import <Foundation/Foundation.h>

@interface SingletonSample : NSObject

// シングルトンパターンであることを定義
+ (SingletonSample *)sharedInstance;

@end
```

```objective-c
// SingletonSample.mファイル
#import "SingletonSample.h"

@implementation SingletonSample

+ (SingletonSample *)sharedInstance
{
	static SingletonSample *sharedInstance = nil;
	static dispatch_once_t onceToken;
	dispatch_once(&onceToken, ^{
		sharedInstance = [SingletonSample alloc] init];]
	});

	return sharedInstance;
}

@end
```

これでシングルトンパターンのクラスとして定義できました。  

次にクラスメソッドとインスタンスメソッドを定義してみます。  

```objective-c
// SingletonSample.hファイル
#import <Foundation/Foundation.h>

@interface SingletonSample : NSObject

// インスタンス変数
@property(assign, nonatomic) NSInteger num;
@property(strong, nonatomic) NSString *str;

// シングルトンパターンであることを定義
+ (SingletonSample *)sharedInstance;

// クラスメソッド
+ (void)testClassMethod;

// インスタンスメソッド
- (void)testInstanceMethod;

@end
```

```objective-c
// SingletonSample.mファイル
#import "SingletonSample.h"

@implementation SingletonSample

+ (SingletonSample *)sharedInstance
{
	static SingletonSample *sharedInstance = nil;
	static dispatch_once_t onceToken;
	dispatch_once(&onceToken, ^{
		sharedInstance = [SingletonSample alloc] init];]
	});

	return sharedInstance;
}

- (id)init
{
	self = [super init];
	
	if(self) {
		// 初期化処理
		self.num = 10;
		self.str = @"テスト";
	}
	
	return self;
}

+ (void)testClassMethod
{
	NSInteger cnt = 20;
}

- (void)testInstanceMethod
{
	self.num = 30;
	self.str = @"TEST";
}

@end
```

さて、この定義したシングルトンのクラスを使ってみましょう。  
適当にUIViewControllerクラスのViewController.mで使ってみます。  

```objective-c
// ViewController.m
- (void)viewDidLoad {
	[super viewDidLoad];

	// インスタンス化
	[SingletonSample sharedInstance];
	// インスタンス変数を確認
	NSLog(@"num: %ld, str: %@", [SingletonSample sharedInstance].num, [SingletonSample sharedInstance].str);
	
	// クラスメソッドの実行
	[SingletonSample testClassMethod];
	// インスタンスメソッドの実行
	[SingletonSample sharedInstance] testInstanceMethod];
	
	// インスタンス変数
	NSLog(@"num: %ld, str: %@", [SingletonSample sharedInstance].num, [SingletonSample sharedInstance].str);]
}
```

ここまで書いた後にRunを実行すると、下記のログが出力されます。  
2014-10-21 22:19:38.922 SingletonSample[4334:26435] num: 10, str: テスト  
2014-10-21 22:19:46.986 SingletonSample[4334:26435] num: 30, str: TEST  

正しく実行されていることがわかりますね。  

このようにクラスメソッドは  
**[クラス名 メソッド名]**  
で呼び出します。  

一方でインスタンスメソッドは  
**[インスタンス メソッド名**  
で呼び出します。  
今回はシングルトンパターンであるため、シングルトンパターンのインスタンスである**[SingletonSample sharedInstance]**を用いていることに注意してください。  

これからシングルトンのクラスをどしどし使いこなしていきたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
