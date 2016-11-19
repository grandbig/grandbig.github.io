---
layout: post
title: "OS X Serverを使ってみよう"
date: 2014-07-12 19:51
comments: true
categories: osx
---

###初めてOSX Serverを使ってみた
本日は少し、趣向を変えてみます。  
筆者はこれまでWeb共有をSitesフォルダを作成してApacheの設定書いてApacheを再起動して...という定番のあれをやっていました。  
しかし、**iOS Developer契約をしている開発者であればOS X Serverを無料で利用できる**と聞いたので、実際に使ってみたくなったのです。  
あまり、ハマる人はいないかもしれないのですが、**定番のWeb共有からOS X Serverに乗り換えたい**という人のために手順をまとめたいと思います。  

<!--more-->

####OS X Serverをインストールしよう
まずは、OS X Serverのインストールから説明したいと思います。通常の手順でApp Storeからダウンロードしようとするとお金がかかるのでご注意を。  
1: Apple Developer サイトにログイン  
[iOS Dev Center](https://developer.apple.com/devcenter/ios/index.action)の下の方を見てください。OS X Serverが見つかるはずです。  
![iOS Dev Center](/images/osx_server1.png)  
OS X Serverの欄の右側にボタンが見えると思いますので、クリックしてApp Storeアプリを起動しましょう。そこで、無料でダウンロードができます。  

2: OS X Serverをインストール  
流れに沿ってインストールしましょう。一応、流れは下記に画像として貼っておきます。  
![続けるをクリック](/images/osx_server3.png)  
![同意する](/images/osx_server4.png)  
![そのまま待つ](/images/osx_server5.png)  
インストールが完了すると、２つの画面が表示されます。  
![チュートリアル画面](/images/osx_server6.png)  
![OS X Server画面](/images/osx_server7.png)  

さて、ここまでは難なく進めると思います。  
冒頭でも述べましたように筆者は定番のWeb共有をやっていましたので、ブラウザで[http://localhost](http://localhost)を見たときに下記のような画面が表示されていました。  
![定番のWeb共有での画面](/images/osx_server2.png)  
しかし、上記の手順2まで終了したタイミングで[http://localhost](http://localhost)を見てみると...  
![ページが見つかりません](/images/osx_server8.png)  
「何かおかしい...!?」てなことになっています。  

案ずるなかれ、OS X ServerでWebサイトの設定をすれば良いのです！！  
3: OS X Server画面で左メニューからWebサイトの項目を選択  
![Webサイト設定画面](/images/osx_server9.png)  
左上のスイッチが切になっているので、入にしましょう。  
![スイッチを入にする](/images/osx_server10.png)  
さあ、改めて[http://localhost](http://localhost)を見てみましょう！！  
OS X Serverのデフォルトページが表示されるようになったはずです。  
![OS X Serverのデフォルトページ](/images/osx_server11.png)  

とは言え、Sitesフォルダに入れていたものが見えなくなっとる...てなことになりますよね？？  
一体、OS X Serverはどこのパスを見ているのか！？  
これはWebサイトの設定画面から確認できます。下記画像のように**サーバ Webサイト**をダブルクリックしましょう。  
![OS X Server Webサイトの設定画面からパスを調べる](/images/osx_server12.png)  
**サーバWebサイト**というタイトルの画面に変わると思います。下記画像を見てもわかるように、赤枠で囲った部分にパスを見ることができます。  
![OS X Server Webサイトの設定画面からパスを調べる](/images/osx_server13.png)  

このパスにこれまでの方法で見ていたファイルを突っ込めば問題なく見れるようになります。  
元々の場所から変えたくないという人はサイトファイルの保存場所を変えれば良いと思います。  

ということで、OS X Serverの基本的な設定が完了したと思います。Webサイトの共有だけでなく、様々な機能をOS X Serverは持っていますので、これからどんどん試していきたいと思います。  
本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
