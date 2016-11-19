---
layout: post
title: "Network Link Conditionerを使ってみよう！"
date: 2016-03-24 21:17
comments: true
categories: ios
---

####iPhoneシミュレータで速度制限してみよう！
少し間が空きましたが、ブログを更新します。  
本日は **iPhoneシミュレータで速度制限する方法** を見ていきたいと思います。  
と言うのも、筆者はこれまでシミュレータでの開発をあまりやったことがありませんでした。  
なぜなら開発端末のスペックが基本的にギリギリだったからです...。  

少しでも開発端末のメモリ消費量を減らすために実機での開発をしてきました。  
しかし、『手元に実機がある環境というのも実は贅沢なのでは？』と思い直し、  
今更ではありますが、本日のお題に行き着いたわけです。  

では早速見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####Network Link Conditionerをインストールする
iPhoneシミュレータには実機と違って、速度制限する機能が設定アプリ内にありません。  
なので、Appleが提供している **Network Link Conditioner** ツールをインストールします。  

0．Apple Developerサイトにサインイン  
まずは、[Apple Developerサイト](https://developer.apple.com/downloads/)にアクセスしましょう。  
開発者IDでサインインしていない場合、下記のようなサインイン画面にリダイレクトされます。  

![Apple Developer Sign in](/images/network-link-conditioner1.png)  

サインイン後、Download画面には飛ばずに、別画面にリダイレクトされます。  

![Apple Top Page](/images/network-link-conditioner2.png)  

1．[Apple Downloadサイト](https://developer.apple.com/downloads/)に遷移してダウンロード  
サインインが完了していれば当初の目的のDownloadサイトに問題なくアクセスできます。  

![Download Page](/images/network-link-conditioner3.png)  

2．Network Link Conditioerをインストール  
続いて、インストールした **Hardware IO Tools** の中から **Network Link Conditioner** をダブルクリックしましょう。  

![Network Link Conditionerをインストール](/images/network-link-conditioner4.png)  

※ 全てのユーザに対してインストールするか聞かれますが、もちろんOKにしましょう！  
(でないと、先に進めないので...)  

3．Network Link Conditionerを起動  
インストール後、 **Network Link Conditioner** が自動的に起動します。  

![Network Link Conditioner](/images/network-link-conditioner5.png)  

4．通信速度を設定  
これで速度制限が設定できるようになったので、早速設定します。  
と言っても実に簡単で **Profile** から該当する通信形式を選択して左下のスイッチをONにするだけです。  

![通信速度の選択](/images/network-link-conditioner6.png)  

上手を見て分かる通り、  
100% Loss, 3G, DSL, Edge, High Latency DNS, Very Bad Network, Wifi  
から選択できます。  

それぞれの意味は下記のようなイメージでしょうか。  

* 100% Loss: Packetが全てロスするので、何も通信できません  
* 3G: モバイル端末での3G通信  
* DSL: アナログ回線による通信方式(有名なのはADSL)  
* Edge: 広い意味での3Gの1つだが、2.75Gとも言われる  
* High Latency DNS: DNSサーバ側のDelayを想定  
* Very Bad Network: 非常に通信状態が悪い  
* Wifi: Wifi環境  

今や、都内では3G環境を探す方がたいへんだったりしますが、  
通信状態が悪いところでの実験は必要不可欠だと思います。  

因みに、2回目以降はシステム環境設定から **Network Link Conditioner** の起動が可能です。  

![システム環境設定から起動](/images/network-link-conditioner7.png)  

ぜひ試してみてください。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
