---
layout: post
title: "Objective-cで改めて同期・非同期について考えてみる"
date: 2014-09-07 21:30
comments: true
categories: ios request
---

###NSURLConnectionで同期・非同期通信をしてみる
さて、1ヶ月ぶりでしょうか....。久々にブログを更新します。  
本日は、もはやObjective-c開発者なら誰しも使うであろうNSURLConnectionでの同期・非同期通信について書きたいと思います。  

NSURLConnectionで同期通信を行う場合、 **sendSynchronousRequest** を利用します。一方、非同期通信を行う場合、 **sendAsynchronousRequest** を利用します。  
この2つの通信のどちらを利用するかについてはまさに **使いドコロ** によります。  
同期通信の場合、メインスレッドでの実行のため、通信先サーバで処理の遅延があると、その間はアプリのUI操作ができなくなります。そのため、通信中はアプリの操作が発生し得ない場合に適していると言えます。  
逆に、通信頻度が多いアプリの場合、UI操作を滞らせることがないように非同期通信を利用するべきです。

<!--more-->

実際に同期・非同期の場合の通信の順番を見てみましょう。  
下記のようなファイル構成を作ります。  

```objective-c
syncAsyncTest					// syncAsyncTest.xcodeprojを作成した際に自動で作られるフォルダ
|
|
L____ViewController				// メイン画面用のUIViewController
		|
		L____ViewController.h
		L____ViewController.m
|
|
L____AppDelegate.h				// 自動で作成されるdelegateヘッダーファイル
L____AppDelegate.m				// 自動で作成されるdelegateモデルファイル

```

続いて、ViewController.mに同期・非同期メソッドを下記のように定義します。  

```objective-c
// 非同期通信メソッド
- (void)sendAsynchronousRequest:(NSURL *)url
{
	NSURLRequest *request = [NSURLRequest alloc] initWithURL:url cachePolicy:NSURLRequestReloadIgnoringLocalCacheData timeoutInterval:10];
	
	[NSURLConnection sendAsynchronousRequest:request queue:[[NSOperationQueue alloc] init] completionHandler:^(NSURLResponse *response, NSData *resData, NSError *error) {
		if(error) {
			NSLog(@"error: %@", error);
		} else {
			NSLog(@"resData: %@", resData);
		}
	}];
}

// 同期通信メソッド
- (void)sendSynchronousRequest:(NSURL *)url
{
	NSURLRequest *request = [NSURLRequest alloc] initWithURL:url cachePolicy:NSURLRequestReloadIgnoringLocalCacheData timeoutInterval:10];

	NSError *error;
	NSURLResponse *response;
	NSData *resData = [NSURLConnection sendSynchronousRequest:request returningResponse:&response error:&error];

	if(error) {
		NSLog(@"error: %@", error);
	} else {
		NSLog(@"resData: %@", resData);
	}
}

// asyncBtnをタップしたときに実行されるアクション
- (IBAction)asyncAction:(id)sender {
	NSURL *url = [NSURL URLWithString:@"通信先サーバのURL"];
	NSLog(@"リクエスト前");
	[self sendAsynchronousRequest:url];
	NSLog(@"リクエスト後");
}

// syncBtnをタップしたときに実行されるアクション
- (IBAction)syncAction:(id)sender {
	NSURL *url = [NSURL URLWithString:@"通信先サーバのURL"];
	NSLog(@"リクエスト前");
	[self sendSynchronousRequest:url];
	NSLog(@"リクエスト後");
}
```

※asyncBtnとsyncBtnの2つのボタンををViewController.xibに配置しています。  

上記のソースを実行すると、  
asyncBtnをタップした場合: リクエスト前⇒リクエスト後⇒sendAsynchronousRequest内のログ  
syncBtnをタップした場合: リクエスト前⇒sendSynchronousRequest内のログ⇒リクエスト後  
といったログの順番になります。  

このように処理に順番を踏まえた上で同期・非同期のどちらを利用するのか決めましょう。  

また、お気づきの方もいるかもしれませんが、  
sendAsynchronousRequestメソッドを利用した場合、それをラッピングしたメソッドでは **sendAsynchronousRequestでサーバから取得してきたデータを戻り値として設定することができません。**  
どういうことかと言うと、  

```
// 非同期通信メソッド
- (NSData *)sendAsynchronousRequest:(NSURL *)url
{
	NSURLRequest *request = [NSURLRequest alloc] initWithURL:url cachePolicy:NSURLRequestReloadIgnoringLocalCacheData timeoutInterval:10];
	
	[NSURLConnection sendAsynchronousRequest:request queue:[[NSOperationQueue alloc] init] completionHandler:^(NSURLResponse *response, NSData *resData, NSError *error) {
		if(error) {
			NSLog(@"error: %@", error);
		} else {
			NSLog(@"resData: %@", resData);
		}

		return resData;
	}];
}
```

と定義すると **Incompatible block pointer types sending 'NSData *(^).....** といったエラーが発生します。  

これはsendAsynchronousRequestの戻り値がvoid型で定義されているにも関わらず、NSData型の戻り値を返そうとしているのでエラーが発生しています。  
特に工夫なく、サーバから取得したデータをラッピングしたメソッドの戻り値として返したいのであれば、sendSynchronousRequestを使えば良いのですが、冒頭で述べたように利用用途が限られます。  

sendAsynchronousRequestメソッドを利用しつつ、その通信で取得したデータを別の処理で使い回したい場合は **自作クラスにDelegateメソッド** を実装することで解決可能です。  

下記のようなフォルダ構成にしてみましょう。  


```
syncAsyncTest					// syncAsyncTest.xcodeprojを作成した際に自動で作られるフォルダ
|
|
L___TestDelegate				// 自作Delegateメソッドを実装するクラス
		|
		L____TestDelegate.h
		L____TestDelegate.m
L____ViewController				// メイン画面用のUIViewController
		|
		L____ViewController.h
		L____ViewController.m
|
|
L____AppDelegate.h				// 自動で作成されるdelegateヘッダーファイル
L____AppDelegate.m				// 自動で作成されるdelegateモデルファイル

```

これまでViewControllerで実装していたsendAsynchronousRequestメソッドをTestDelegateクラスに移行します。  

TestDelegate.hは下記  

```objective-c
#import <Foundation/Foundation.h>

@protocol TestDelegate <NSObject>
@optional

- (void)successMethod:(NSData *)data;
- (void)failedMethod:(NSError *)err;

@end

@interface TestDelegate : NSObject

@property (nonatomic, assign) id<TestDelegate> delegate;

- (void)sendAsynchronousRequest:(NSURL *)url;

@end
```
続いて、TestDelegate.mは下記  

```
#import "TestDelegate.h"

@implementation TestDelegate

- (void)sendAsynchronousRequest:(NSURL *)url
{
	NSURLRequest *request = [NSURLRequest alloc] initWithURL:url cachePolicy:NSURLRequestReloadIgnoringLocalCacheData timeoutInterval:10];
	
	[NSURLConnection sendAsynchronousRequest:request queue:[[NSOperationQueue alloc] init] completionHandler:^(NSURLResponse *response, NSData *resData, NSError *error) {
		if(error) {
			if ([self.delegate respondsToSelector:@selector(failedMethod:)]) {
				NSLog(@"error: %@", error);
				[self.delegate failedMethod:error];
			}
		} else {
			if ([self.delegate respondsToSelector:@selector(successMethod:)]) {
				NSLog(@"resData: %@", resData);
				[self.delegate successMethod:resData];
			}
		}
	}];
}

```

これでDelegateメソッドの実装とsendAsynchronousRequestメソッドの移行が完了しました。  

Delegateメソッドを呼び出す側のViewController.mは下記のようになります。  

```
#import "ViewController.h"
#import "TestDelegate.h"

@interface ViewController () <TestDelegate>

@property (strong, nonatomic) IBOutlet TestDelegate *td;

@end

<省略>

- (void)viewDidLoad
{
	[super viewDidLoad];
	
	self.td = [TestDelegate alloc] init];
	self.td.delegate = self;]
}

<省略>

// asyncBtnをタップしたときに実行されるアクション
- (IBAction)asyncAction:(id)sender {
	NSURL *url = [NSURL URLWithString:@"通信先サーバのURL"];
	NSLog(@"リクエスト前");
	[self.td sendAsynchronousRequest:url];
	NSLog(@"リクエスト後");
}

// TestDelegateクラスで実装したsuccessMethod
- (void)successMethod:(NSData *)data
{
	    NSLog(@"data: %@", data);
}

// TestDelegateクラスで実装したfailedMethod
- (void)failedMethod:(NSError *)err
{
	    NSLog(@"err: %@", err);
}
```

これでTestDelegateクラス内のsendAsynchronousRequestを実行して取得したデータをViewControllerクラス内のsuccessMethodやfailedMethodで受け取ることができるようになりました。  

今回改めて調べてみることで、通信の同期・非同期は意外と奥が深いということがわかりました。  
ということで今日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

