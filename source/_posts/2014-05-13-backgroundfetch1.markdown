---
layout: post
title: "Background fetch completionHandlerの実行タイミングについて"
date: 2014-05-13 22:51
comments: true
categories: ios BackgroundFetch
---

###Background Fetchで気をつけること
本日はBackground fetchについて書きたいと思います。時間の都合上、何回かに分けて書く予定です笑  
Background fetchは既に皆さんもご存知の通りiOS7から加わった新機能です。通常、アプリをBackground起動にしたとき、アプリ内処理に許される時間は5秒間です。iOS6まではBackground起動中に継続して処理可能な機能は決まっていました。(位置情報の取得, 音楽の再生, 電話の3つ)  
従って、これまでは**beginBackgroundTaskWithExpirationHandler**と**setKeepAliveTimeout**を用いて最大10分間の処理を継続して実行していました。(しかし、iOS7ではこの方法ができないらしいです。)  
これに対してBackground fetchは30秒の制限はあるものの、OSが実行すべきと思ったタイミングで複数回実行されます。継続して実行する必要はないものの、どこかのタイミングで実行しておきたいというときには有効に使うことができるでしょう。  

<!--more-->

まず、『Background fetchの利用を許可するまでの手順』などの基本的な部分についてはいろいろなサイトで説明されているので、例えば[こちら](http://blog.yug1224.com/2013/09/29/backgroundfetch/)のサイトなどを参考にして下さい。  
では早速、本日のお題である**completionHandlerの実行タイミング**の説明です。これはBackground fetchの処理が終了したことをiOSに知らせるために必ず書く必要があります。もし、書かなかった場合はデバッグ時に  
**Warning: Application delegate received call to -application:performFetchWithCompletionHandler: but the completion handler was never called.**  
というWarningメッセージが表示されます。  
また、処理が終了したことをiOSが判断できないため、  
**permittedBackgroundDuration: 30.000000 reason: backgroundContentFetching owner pid:33 preventSuspend  preventThrottleDownUI  preventIdleSleep  preventSuspendOnSleep**  
とアプリ起動中にエラーを出すことになります。(これはOrganizerのDebug Logsから確認することができます。)  

そして重要なのは**completionHandlerは処理の最後に書く**ということです。  
しかも**一本筋のフローの中での最後**という意味です。  
例えば、  
AppDelegate.mファイル  
```objective-c
- (void)application:(UIApplication *)application performFetchWithCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
{
	[self testMethod];

	completionHandler(UIBackgroundFetchResultNewData);
}

- (void)testMethod
{
	dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
	dispatch_async(globalQueue, ^{
		NSLog(@"別スレッド処理の開始");
		[NSThread sleepForTimeInterval:5.0f];
		NSLog(@"別スレッド処理の終了");
	});
}
```
このような処理があったとします。  
この場合、デバッグしてみるとわかるのですが、testMethodの最後の**NSLog(@"別スレッド処理の終了");**は実行されません。  
これは先にcompletionHandler(UIBackgroundFetchResultNewData);が実行されてしまうためです。  

そういった意味で**一本筋のフローの中での最後にcompletionHandlerの処理を書く**ということです。  

このようにBackground fetchの実行タイミングは非常に重要になります。  
といったところで本日はここまで。  

参考:  
[Background fetchを試してみた](http://qiita.com/griffin_stewie/items/8371c09059b3ba7bb202)  
[iOSでバックグラウンドタスク](http://miyano-harikyu.jp/sola/devlog/2013/11/21/post-108/)  
[iOS7のバックグラウンド処理〜Background Fetch](http://wonderpla.net/blog/engineer/BackgroundFetch/)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
