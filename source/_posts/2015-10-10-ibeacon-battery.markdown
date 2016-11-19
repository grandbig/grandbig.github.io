---
layout: post
title: "iBeaconはバッテリを使う？使わない？"
date: 2015-10-10 22:26
comments: true
categories: ios iBeacon
---

####iBeaconのバッテリ消費量を調査しよう
さて、今回はiBeaconを利用したサービスを始める上でネックとなる(正確にはネックと思われがちな)バッテリの消費量について書きたいと思います。  
iPhoneでiBeaconを利用する場合、 **Bluetooth** だけでなく、 **位置情報サービス** をONにする必要があります。  

![iBeaconを使うための設定](/images/ibeacon-setting1.png)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

一般的なユーザのイメージだと、スマートフォンでなるべくバッテリ消費量を抑えたいのであれば、『 **Bluetooth** や **位置情報サービス** をOFFにすべし！！』というのが定説になっていることかと思います。  
しかし、開発者からしてみれば、 **Bluetooth** によるバッテリ消費には疑問があります。というのも現在多くのユーザが持っているスマートフォンでは、搭載されているBluetoothが4.0以上になっていることも珍しくありません。  
であるならば、ほとんどのユーザは超低消費電力の **BLE** の恩恵を受けており、Bluetoothによる電池の消費は誤差の範囲と言えます。  

だがしかし、幾ら開発者が『 **Bluetooth** はバッテリを消費しません！』と声高らかに訴えたとしても、ユーザからは『自らのサービスを利用してもらうための体の良い発言』と捉えられてしまうかもしれません。  
であるならが、開発者にできることはただ一つ、データで数値で示すということではないでしょうか？  
ということで今回はXcodeによるバッテリ消費量の計測を交えながらiBeaconのバッテリ消費量に迫っていきたいと思います。  

####Xcodeでアプリのバッテリ消費量を計測しよう
ではXcodeを使ったアプリのバッテリ消費量の計測方法を紹介したいと思います。  
まずは、計測したいiPhone側で設定項目を確認しましょう。  

1. 設定アプリを起動
2. デベロッパを選択
3. Loggingを選択
4. Start Recordingを選択

※ 1度でも開発用のアプリをビルドしてインストールしていれば『デベロッパ』の項目が出るはずです。  

![iPhoneの設定](/images/ibeacon-setting2.png)  

でなければ、  
Xcode > Product > ProfileからiPhoneにアプリをインストールしましょう。  

![Xcode > Product > Profile](/images/ibeacon-setting3.png)

これで計測が開始されるので、計測終了時に同じ手順で **Stop Recording** を選択しましょう。  
計測が完了したら、Xcodeで取込みを開始します。  

Xcode > Product > ProfileでInstrumentsを起動します。  
その中の **Energy Diagnostics** を選択します。  

![バッテリ計測にはこれ！](/images/ibeacon-setting4.png)  

すると、下図のようなウィンドウが表示されます。  

![Energy Diagnosticsウィンドウ](/images/ibeacon-setting5.png)  

####iBeaconの結果について
ここまでは計測方法について説明しました。  
では、iBeaconを使った場合の結果は結局どうなうのでしょうか？  

もちろん調べましたので、さくっとご紹介します。  

![iBeaconを使った時のバッテリ消費について](/images/ibeacon-setting6.png)  

【結果】  

* iBeaconはGPSを利用しない
上記グラフを見ると、Bluetoothはオレンジ色になっていますが、GPSは緑色のままだと思います。オレンジはONを表し、緑はOFFを表します。  
* バッテリ消費率は ディスプレイ > 通信 > Bluetoothの順に消費率が高い
Energy Usage Levelを見ると、Display Brightness Levelが反応しているときが最も高く、その次にNetworkが反応しているときというのがわかります。  

この結果からもわかるように、iBeaconはGPSを利用していないんです！  
因みに、Bluetoothだけで言えば、1時間あたり1%も利用しません。Bluetoothを介して音楽を聞いたり、3G/4G/Wi-Fiなどで秒間何らかの通信をしたりすれば、バッテリは消費しますが、Bluetoothが原因というよりも、Bluetoothをトリガーとしたやり取りが原因ということなんですね。  

このことがユーザにもっと浸透する＆開発者がBluetoothの誤解を広めないようにすることができれば、世の中でiBeaconを利用したサービスがユーザに様々なメリットを提供してくれるのだと期待しています。  
といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
