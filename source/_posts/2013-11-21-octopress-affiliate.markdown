---
layout: post
title: "Octopressでアフィリエイト！"
date: 2013-11-21 22:40
comments: true
categories: octopress アフィリエイト
---

###Octopressでアフィリエイトを始めよう！！
さて、本日は久々にOctopressのことについて紹介させて頂きます。  
前々からアフィリエイトはやろうと決めていたのですが、億劫で進んでいませんでした。  
そして、いざやってみたところ、  
**Octopressへの載せ方がわからない！！**  
**ネットで調べてもよくわからない！！！**  
なんてことがあったのでメモがてら書こうと思った次第です。

<!--more-->

####アフィリエイト広告を表示しよう
筆者はとりあえず、**iTunesアフィリエイト**と**Google Adsense**に登録してみました。iTunesアフィリエイトは1営業日で審査が通りました。Google Adsenseは1次審査まで1営業日, 2次審査は広告をブログに貼ってから1営業日で通過しました。  
まず、Google Adsenseですが、広告を作成すると下記のように広告を表示するためのコードを取得することができます。  
![Google Adsenseで広告コードを取得](/images/octopress_affiliate1.png)  

これをどこに書き込むかで表示できる場所が異なります。  
1: サイドに表示したい場合  
2: ブログの途中に表示したい場合  

**1: サイドに表示したい場合**  
まず、octopress/source/\_includes/custom/asides/ 配下にhtmlを作成します。  
筆者はgoogleAdsense.htmlを作成しました。  
次にそのhtmlを編集して、取得したコードをsectionタグ内に書きます。  

```
<section>
<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- 広告１ -->
<ins class="adsbygoogle"style="display:inline-block;width:300px;height:250px"data-ad-client="*******************"data-ad-slot="**********"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
</section>
```

そして、octopress/_config.ymlを編集します。default_asidesの配列内に自作したhtmlを追記する。  
```
default_asides: [asides/recent_posts.html, asides/github.html, asides/delicious.html, custom/asides/googleAdsense.html]

```

これでOKです。Google Adsenseは2次審査が完了するまで空白が表示されるため、正しくできているかどうかは  

rake previewを実行  
ブラウザでプレビュー画面にアクセス  
表示されるであろうサイドの辺りを右クリックしてデバックコンソールを開く  
先ほど書き込んだソースがあることを確認  

ここまで確認が取れれば、審査終了後、すぐに表示されます(一応、48時間以内と言われていますが…)。  

**2:ブログの途中に表示したい場合**  
例えば下記のようにブログの途中に広告を表示したい場合はどうすれば良いかというと、  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- 広告2 -->
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

ただ単に、取得したコードを表示したいmarkdownファイルに書くだけです。  
ただし、tabが入っている場合はmarkdownの書式として判別されてしまうので注意しましょう。  

iTunesアフィリエイトでも下記のようにソースコードを取得することができます。(iTunes Link MakerでiOS Appのリンクを作成)  
![iTunes Link Makerでリンク作成](/images/octopress_affiliate2.png)  

これをGoogle Adsenseのときと同様に貼り付けるとリンクを貼ることができます。  
(筆者は過去のアプリ紹介ページにリンクを貼ってみました。)

他にもいろんなアフィリエイトを試してみようかな！ということで本日はここまで。
