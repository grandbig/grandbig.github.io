---
layout: post
title: "東京Node学園付属小学校 1限目に参加してみた！"
date: 2016-01-15 19:27
comments: true
categories: node study
---

####Node.jsを基礎から見直そう！
以前、業務でNode.js v0.6.12だったり、v0.10.24だったりを触ってきました。  
しかし個々最近は、ネイティブアプリ開発にかかりっきりになっていたので、Node.jsから遠ざかってしまっていました。  
そんなこともあり、「これじゃいかん！」ということで今回、 **東京Node学園付属小学校 1限目** に参加してみることにしました。  

通常のNode学園は行ったことがあったものの、非常にレベルが高いなと感じた経験があり、まずは思い出す足掛けということで...。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####勉強会の内容と感想
さて、勉強会の内容と感想について書いていきます。  

#####オープニング
teratailでは技術の壁をなくしたい。  
何かサービスを開発するときには、いつも使っている言語を選びがちだけど、一番適した技術を使ったり、新しい技術を導入したりできると良いな〜という感じ。  

工数やリソースの問題から、保守的になって実績のあるやり方で進めるパターン多いんだよな〜と思いながら聞いていました笑。  

#####ゲスト講演
ゲスト講演はNode.js日本ユーザーグループ代表の古川陽介さんが担当されていました。  
内容はあえてNode.jsではなく、Guillermo Rauch(socket.ioの作者)が提唱した  **7 principles of rich web applications** でした。  
筆者は勉強不足のせいもあって、初めて知りました。  

古川さん曰く、実際に7つの原則を満たしているWeb Applicationは存在しないんではないかと...。  
詳しい内容は[【翻訳】リッチなWebアプリケーションのための7つの原則](http://yosuke-furukawa.hatenablog.com/entry/2014/11/14/141415)に書かれています。  

筆者個人としてはSPA(Single Page Application)が興味を惹いている理由を知れたり、[RAIL](https://developers.google.com/web/tools/chrome-devtools/profile/evaluate-performance/rail)という重要な指標を知れたりと、かなり勉強になりました。  

また特に印象的だったのは、 **レイテンシの壁** の話でした。  
通信をする限り、レイテンシは必ず存在するのだと。  
それを処理の仕方で早く見せることができないかと鎬を削る様がとても惹きつけられました。  

最近ネイティブ開発ばかりやっていた自分としては、描画に対するレイテンシはあまり気にしていませんでした。  
きちんと考えないとダメだなと。  

JavaScriptは7つの原則をクリアできるのだ！！

#####LT
続いて、LTです。  
**東京Node学園付属小学校** という名にふさわしく、知識・技術的についていけないといったことなく、落ち着いて聞くことができました。  

以下、聞いたことを箇条書き  

- お役立ちツール  
    - [SWAGGER](http://swagger.io/)
Web APIドキュメント生成ツール
    - [Node-RED](http://nodered.org/)
Drug&Dropで簡単にWebアプリを作れるツール
    - [LoopBack](http://loopback.io/)
Node.jsベースのBaaSを作れるオープンソース
　
- Node.jsが使われているサービス
    - Uber
    - PayPal(JavaからNode.jsに移行)
    - NETFLIX
　
- Node.jsの良いところ
    - 非エンジニアでも簡単にWebアプリが作れる
    - クライアントとサーバサイド共にJavaScriptで書ける
    - リアルタイム通信でいろいろなWebアプリを作りたくなる
    - IoTでも使える
    - 「同時に接続して、チームで協力してゴールを達成する」方向にも注目...

とりあえず、Node.js意欲が湧いてきたので、何か作りたい！！  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
