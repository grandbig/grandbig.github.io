---
layout: post
title: "AplixのMyBeaconを使ってみた"
date: 2014-05-03 22:48
comments: true
categories: ios iBeacon
---

###AplixのUSB給電型Beacon発信機 MyBeaconを使ってみた
先日、Aplix社が発売を開始したMyBeaconを早速購入して試してみました。  
まずはAplix社の提供している専用のiOSアプリケーションをダウンロードしてみましょう。  
<a href="https://itunes.apple.com/jp/app/mybeacon-tool/id848908053?mt=8&uo=4&at=11lpfc" target="itunes_store">MyBeacon Tool - Aplix</a>

次にMyBeaconの送付物の中に同封されていた紙に書かれている[URL先](http://www.aplix.co.jp/beacon/mb001_support/)にアクセスしてみましょう。  
すると、下記の画面が表示されます。  
![Aplix社のMyBeacon用の画面](/images/aplix_mybeacon_1.png)  

<!--more-->

ここで**管理者情報確認**を選択しましょう。  
遷移先画面では**メールアドレス**と**プロダクトキー**の入力が求められます。ここには先ほどと同じ同封された紙に書かれた**お客様情報**と**プロダクトキー**を入力してください。  
![Aplix社のMyBeacon用の画面2](/images/aplix_mybeacon_2.png)  

結果、**Proximity UUID**と**MyBeaconToolアプリで使用するPassword**を知ることができます。  
![Aplix社のMyBeacon用の画面3](/images/aplix_mybeacon_3.png)  

さて、ここまでわかったところでMyBeaconToolアプリを起動してみましょう。すると、MyBeaconIDとパスワードの入力を求められます。  
![Aplix社のMyBeacon用の画面4](/images/aplix_mybeacon_4.png)  

入力が終わったら、MyBeaconのサーチが開始できます。MyBeaconをコンセントに刺してセットしてから、**サーチ開始**してみましょう。すると、ひっかかったMyBeaconの情報が表示されます。  
![Aplix社のMyBeacon用の画面5](/images/aplix_mybeacon_5.png)  

以上で、**Proximity UUID**，**Major**，**Minor**の値が判明しました。  
実際にBeacon機はMyBeaconを使えば良いので、**Proximity UUID**さえわかれば問題ありません。  
自作アプリ内のソースでは下記のように**Proximity UUID**に設定しましょう。  
```objective-c
NSUUID *proximityUUID = [[NSUUID alloc] initWithUUIDString:@"ここにProximity UUIDを書きましょう。ハイフン含めて下さい"];
```

結果、距離を測ったりできました。  
![自作アプリで距離を計測できました](/images/aplix_mybeacon_6.png)  

これで簡単にiBeaconの自作アプリが作れますね。皆さん、安価ですし、この機会にMyBeaconを買ってみてはいかがでしょうか。

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

