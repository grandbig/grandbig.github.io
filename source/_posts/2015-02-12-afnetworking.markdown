---
layout: post
title: "AFNetworking2.xでWeb-APIを叩いてみた！"
date: 2015-02-12 00:01
comments: true
categories: ios api
---

###AFNetworking2.x
さて、本日は今更ながら超有名ライブラリである **AFNetworking** を使ってみたいと思います。  
ただ、単に使い方を説明するのであれば、他のサイトにお任せすれば良いので、Web-APIを使ってみる形で書きたいと思います。  

今回は、  
GETリクエスト: **OpenWeatherMap API**  
POSTリクエスト: **VoiceText Web API**  
を試してみました。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->
　

**AFNetworking** を使うためにはCocoaPodsでライブラリをインストールして、  
ViewController.mで`#import "AFNetworking.h"`を書きましょう。  

まずは、OpenWeatherMap APIを使って、GETリクエストを投げてみました。  

```objective-c
// ViewController.m
#import "ViewController.h"
#import "AFNetworking.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];

	// AFHTTPSessionManagerをインスタンス化
	AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];

	// OpenWeatherMap APIを利用
	[manager GET:@"http://api.openweathermap.org/data/2.5/weather?q=Tokyo,jp"
	  parameters:nil
	     success:^(NSURLSessionDataTask *task, id responseObject) {
			 NSLog(@"response: %@", responseObject);
		 } failure:^(NSURLSessionDataTask *task, NSError *error) {
			 NSLog(@"error: %@", error);
		 }
}
```

きちんと値が返ってきました。  

続いて、[VoiceText Web API](https://cloud.voicetext.jp/webapi)を使って、POSTリクエストを投げてみました。  
VoiceText Web APIは無料利用に登録が必要なので、さくっと登録しましょう。  
すぐに登録したメールにAPIキーが来ます。  
これを使って、POSTリクエストを投げてみました。  

```objective-c
// ViewController.m
#import "ViewController.h"
#import "AFNetworking.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];

	// AFHTTPSessionManagerをインスタンス化
	AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];

	// APIキーをセット
	NSString *apiKey = @"<メールで届いたAPIキー>";
	[manager.requestSerializer setAuthorizationHeaderFieldWithUsername:apiKey password:nil];

	// リクエストパラメータをセット
	NSMutableDictionary *params = [NSMutableDictionary alloc] init];
	NSString *text = @"おはようございます。";
	[params setObject:text forKey:@"text"];
	[params setObject:@"show" forKey:@"speaker"];
	[params setObject:@"ogg" forKey:@"format"];]

	// Content-Typeをセット
	[manager setResponseSerializer:[AFHTTPResponseSerializer serializer]];
	manager.responseSerializer.acceptableContentTypes = [NSSet setWithObject:@"audio/ogg"];

	// OpenWeatherMap APIを利用
	[manager POST:@"https://api.voicetext.jp/v1/tts"
	   parameters:params
	      success:^(NSURLSessionDataTask *task, id responseObject) {
			  NSLog(@"response: %@", responseObject);
		  } failure:^(NSURLSessionDataTask *task, NSError *error) {
			  NSLog(@"error: %@", error);
		  }
}
```

注意点は下記です。  

* APIキーの設定
VoiceText Web APIではAPIキーを利用する必要があるので、`requestSerializer`の`authorization`にユーザネームとしてAPIキーを設定しましょう。  
* `Content-Type`の設定
`audio/ogg`でレスポンスを受け取る必要があるので、`responseSerializer`を設定しましょう。  

あとは通常の`NSURLSession`や`NSURLConnection`のPOSTリクエストと同様にリクエストパラメータをセットすれば良いのです。
正しく値が返ってきますね。  

AFNetworkingの良いところはエラーでも値が返ってくるところでしょうか。  
`Content-Type`が対応していないので`success`を通らないということが一目瞭然でした。  

今後は、この便利なライブラリを積極的に使っていこうかと思います。  
といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
