---
layout: post
title: "Google Maps SDK for iOSをiOS6対応させようとしてハマったこと"
date: 2014-01-27 23:50
comments: true
categories: ios map
---

###iOS6からGoogle Maps SDKが使いたい！！
さて、最近取り組んでいるGoogle Maps SDKについて書きます。前回ご紹介させて頂いたときはiOS7からの対応として書かせて頂きました。  
今日はiOS6から対応するアプリを開発する上でハマったことについて説明します。  

Xcode5でiOS6から対応に変更するとき、**Deployment Target**を6.0に変更すると思います。そのままClearn ⇒ Buildをしてみると下図のようなおびただしい数のエラーが...  
![iOS6でビルドするとエラーがたくさん出る](/images/googlemapssdk_howtouse1.png)    

<!--more-->

なぜ、Deployment Targetを変えただけでこんなにもエラーが？と思ったので、エラー内容を見てみると...  
**Undefined symbols for architecture armv7**と書いてあります。  
これだけで調べてもよくわかりませんでした。(原因がわかってからよくよく見てみると様々なサイトでヒントを教えてくれていたとわかるのですが...)  

もう少し、詳しく見てみると**Google Maps**がひっかかっていることがわかりました。で、再度検索し続けると  
**ライブラリが足りない**  
という可能性にたどり着きました。  
さ、[Google Maps SDK 公式ページ](https://developers.google.com/maps/documentation/ios/start?hl=ja)を見てみましょう。  
自分で作成したXcodeプロジェクトに追加してあるライブラリと公式ページに書いてある追加すべきライブラリを比較すると下記の違いに気づきました。(作成したXcodeプロジェクトにはデフォルトで追加されているライブラリも含まれています。)  
![ライブラリが違う](/images/googlemapssdk_howtouse2.png)  

結果、ライブラリが異なるという単純な理由だったわけですが、気づくのに時間がかかりました。  
iOS7では動いてしまうという部分も混乱を招く原因だったと思います。  

ま、英語を嫌がらずに公式ページを読んだ方が時間はかかりませんってことですね...。  
今日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
