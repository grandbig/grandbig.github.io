---
layout: post
title: "BLEメジャー ver 1.0.0"
date: 2013-11-17 21:21
comments: true
categories: ios MyApp iBeacon
---

###iPhoneアプリ『BLEメジャー ver1.0.0』の紹介
本日(2013/11/17)、iPhoneアプリ**BLEメジャー**をリリースしました。本アプリはiOS7から新たに使用可能となったiBeaconの技術を利用した距離計測アプリです。今回はこのアプリの使い方について説明したいと思います。

興味を持った方はインストールページヘ！！  
<a href="https://itunes.apple.com/jp/app/blemeja/id741597546?mt=8&uo=4" target="itunes_store"style="display:inline-block;overflow:hidden;background:url(https://linkmaker.itunes.apple.com/htmlResources/assets/ja_jp//images/web/linkmaker/badge_appstore-lrg.png) no-repeat;width:135px;height:40px;@media only screen{background-image:url(https://linkmaker.itunes.apple.com/htmlResources/assets/ja_jp//images/web/linkmaker/badge_appstore-lrg.svg);}"></a>

<!--more-->

####BLEメジャーの使い方
早速、本アプリの使い方を説明します。  
まず、iOS7以上の端末(iPhone4S以降, iPad 第3世代以降, iPad mini, iPod touch 第5世代以降)を2台用意してください。2端末間の距離を計測するために必要となります。  
次に各端末で**BLEメジャー**アプリを起動します。すると下記画面が表示されます。  

![BLE初期画面](/images/ble_measure1.PNG)  

２台のうちどちらかの端末で**Attract**を、もう片方の端末で**Keep**を選択してください。
![別々のボタンをタップする](/images/ble_measure2.png)  

それぞれ別の画面が表示されます。  
![Attract画面とKeep画面が表示されます](/images/ble_measure3.png)  

Attract(左側の)画面では初期起動時のみ位置情報サービスの利用の可否を求められます。これは必ず許可を選択してください。2端末間の計測が始まっていない時は**default**と表示されます。Keep(右側の)画面では特に操作はありません。  
計測が開始されるとAttract側で数値が表示されます。  

![計測開始](/images/ble_measure4.PNG)  

※うまく計測できない場合は再度、両端末でアプリを起動し直してください。  
※途中、計測がうまくいかなくなったときは-1.000000[m]と表示されますが、そのまま使って頂くと計測が再開されます。
