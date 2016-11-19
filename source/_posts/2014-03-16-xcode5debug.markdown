---
layout: post
title: "Xcode5でDebug gaugesを使ってみよう！"
date: 2014-03-16 21:56
comments: true
categories: xcode
---

###Xcode5のデバッグ機能を使ってCPUとスレッドを調べよう
最近、アプリを開発する上で、その操作が**どのくらいCPUを消費するのか**, **マルチスレッドを有効に使えているのか**を厳しく見る必要が出てきました。それまでCPUやスレッドもそこまで気にしたことはなかったのですが、Xcode5からはInstrumentを起動しなくとも、簡易的に確認できるということで試しにやってみました。  

<!--more-->

####Xcode5でDebug gaugesを使ってみる
まずは、本当に簡単な何もしない画面を作成して、様子を見てみました。  
![何もしない画面の様子](/images/xcode_debug1.png)  

4つスレッドがあります。そして、画面の描画に使われているであろうスレッドがThread1であることがわかると思います。他は全く使ってないですね。  
また、本試験ではiPhone5(iOS7.1)を利用しているため、マルチコアなので、CPUのメータが最大200%であることが示されています。  
しばらく見ていると、4つあったスレッドが2つに減ります。  
![スレッドが2つになる](/images/xcode_debug2.png)  

続いて、for分で10000回ループしてひたすらログを出力するボタンを作成して、様子を見てみました。  
![for分で10000回ループ](/images/xcode_debug3.png)  

さて、スレッドを別に作成して実行してみましょう。  
```objective-c
dispatch_queue_t gQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

dispatch_async(gQueue, ^(void) {
	for(NSInteger j=0; j < 10000; j++) {
		NSLog(@"%d回目のループ", j);
	}
});
```

その結果が下図。  
![別スレッドで実行](/images/xcode_debug4.png)  
これを見るとメインスレッドとは別のスレッドで実行されていることがわかります。基本的に描画はメインスレッドでしか実行がされないので、その他の処理はメインスレッドとは別スレッドで行うのが良さそうですね。  

実際にやってみましょう。  
```objective-c
dispatch_queue_t gQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

dispatch_async(gQueue, ^(void) {
	for(NSInteger j=0; j < 10000; j++) {
		NSLog(@"%d回目のループ", j);
		dispatch_async(dispatch_get_main_queue(), ^(void) {
			NSLog(@"メインのループ");
		});
	}
});
```
![マルチスレッドでの実行](/images/xcode_debug5.png)  
メインスレッドと別スレッドで各10000回ログを出力する処理を実行しているため、同じ時間帯にリソースを使っていることがわかりますね。  

###マップを描画してDebug gaugesを見てみる
アプリを開発するときにマップを利用すると様々な操作でメインスレッドを利用していることがわかります。まずはマップを起動してみましょう。  
![マップを起動してみる](/images/xcode_debug6.png)  
起動直後から大量のスレッドが作成されていることがわかります。描画はメインスレッドでやるものの、様々な計算は別スレッドでやっているということなんでしょうか...。  
起動してからしばらく放置しておくとスレッドがどんどん減っていきます。  
しかし、マップを拡大・縮小するとスレッドが大量に作成されます。また、CPUも激しく消費することがわかります。(余裕で140〜150%まで上がったりしますね...)  

これらはユーザの操作によって引き起こされる現象なので、極端な話、触らなければCPUの消費はされません。  
では、下記のようにしてみたらどうなるでしょうか？  
```objective-c
- (void)mapViewDidFinishLoadingMap:(MKMapView *)mapView
{
	[self.map setUserTrackingMode:MKUserTrackingModeFollowWithHeading animated:YES];
}
```
![ユーザの方向にマップが回転する](/images/xcode_debug7.png)  
ユーザの動きを検知して自動で回転するので、CPUを使い続けます。  

###まとめ
アプリを開発する上で、端末のリソースをどのようにうまく使っていくのかは非常に重要です。マップを使うアプリを作成するときは特に注意が必要ですね。
