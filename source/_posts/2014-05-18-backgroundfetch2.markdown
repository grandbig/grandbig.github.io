---
layout: post
title: "Background FetchとBackground Transferについて"
date: 2014-05-18 21:43
comments: true
categories: ios BackgroundFetch
---

###Background FetchとBackground Transferの違いについて
今回はBackground FetchとBackground Transferについて筆者が混同していたことについて書こうと思います。Background Fetchのできることや使いどころは[前回](http://grandbig.github.io/blog/2014/05/13/backgroundfetch1/)書きましたので、そちらをご覧下さい。  
もともと筆者がBackground Transferに着目した理由はBackground Fetchでは不可能な長い時間の処理を実現させたいからでした。Background Transferでは見事にその想いを実現できる！！っと思っていました。  
しかし、どうやら根本的に理解が間違っていたようなのです...。  

<!--more-->

Background Transferは**時間制限なし**, **アプリケーションの状態によらない転送が可能な設定がある**という特徴があります。そして設定可能なタスクは  
・NSURLSessionDataTask: Background起動時には使えないのでちょっとした通信に利用  
・NSURLSessionDownloadTask: 大容量のダウンロード処理だけでなく、処理時間が長い可能性のある通信でも利用可能  
・NSURLSessionUploadTask: 大容量のアップロード処理で利用  
の3つがあります。  

筆者はBackground Fetchの30秒制限では実行不可能な長めの処理をBackground Transferで実行しようとしたのですが、これがなかなかうまくいかないんですね。**1回の実行**であれば、Background Transferで問題ありません。ただ、**繰り返し処理を実行**したいのであればBackground Transferは向いていません。というかそういった処理は組めないようなんです...。  

Background fetchとBackground Transferを組み合わせた場合、下記のような処理になるでしょう。  
AppDelegate.mファイル内のコード  
```objective-c
- (void)application:(UIApplication *)application performFetchWithCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
{
	NSURLRequest* request = [NSURLRequest requestWithURL:[NSURL URLWithString:@"http://localhost:3000"]];
	NSURLSessionDownloadTask* downloadTask = [session downloadTaskWithRequest:request];
	[downloadTask resume];

	completionHandler(UIBackgroundFetchResultNewData);
}

-(void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didFinishDownloadingToURL:(NSURL *)location
{
	// 通信結果を受け取った後に処理を書くメソッド
	
}
```
これを実行すると、completionHandlerでBackground fetch内での処理が終了したと見なされ、didFinishDownloadingToURLを通過しませんでした。  
よって、アプリがBackground起動のときにOSが判断したちょうど良いタイミングで実行してくれるBackground Fetchではやはり長い処理を実行することはできないと言えます。  
確かにBackground Transferで検索して出てくるサンプルは大体、ボタンを押した後に実行される感じだったんですよね...。  

もしかしたら方法があるかもしれませんが、今のところ手詰まりな感じです。ってことで今日はここまで。  

参考:  
[第 1 回・iOS7でバックグラウンド実行＆通信をしてみよう！](http://www.gaprot.jp/pickup/ios7/vol1/)  
[iOS 7のための開発ノウハウ #2：大容量ファイルのバックグラウンド転送](http://mediaprobe.co.jp/blog/clips/2014/01/09/ios-7-2/)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
