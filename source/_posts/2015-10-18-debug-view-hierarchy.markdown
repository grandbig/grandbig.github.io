---
layout: post
title: "Debug View Hierarchy"
date: 2015-10-18 23:54
comments: true
categories: ios
---

####Debug View Hierarchyを使ってみよう！
今日はDebug View Hierarchyの使い方を見てみたいと思います。  
普段、めったにこの機能を使うことのない筆者ですが、使いどころを探る上でも少し見てみようと思います。  

#####Debug View Hierarchyとは
Xcodeでデバッグした後に、アプリの画面の構造を確認することができる機能です。  
デバッグ後に表示されるバーの下図の赤枠で囲った部分をクリックしましょう。  

![Debug View Hierarchyの開始方法](/images/debugViewHierarchy1.png)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

すると、下図のような表示が見れるはずです。  

![Debug View Hierarchyの初期図](/images/debugViewHierarchy2.png)  

ここでは下図のツールバーを用いて、Debug View Hierarchyを使いこなしていきます。  

![Debug View Hierarchyのツールバー](/images/debugViewHierarchy3.png)  

左から  

* Show clipped content  
紐付けられている画面の構造がわかる
* Show constraints  
各画面や部品に設定されたConstraintsがわかる
* Adjust the view mode  
Contents / Wireframes / Wireframes and Contentsを切り替えられる
* Orient to 2D  
2D / 3D表示を切り替えられる
* Zoom out  
Debug View Hierarchyの表示をズームアウトする
* Actual size  
Debug View Hierarchyを実際のサイズにズームする
* Zoom in  
Debug View Hierarchyの表示をズームインする

となっています。  

#####Debug View Hierarchyの使いどころ
筆者が躓いたときに使えるなと思ったパターンを列挙します。  

1. WKWebViewとネイティブ部品(ボタン, ラベルなど)を同一の画面に配置したとき
WKWebViewはstoryboardから配置できず、ソースコードを書く必要があります。  
ネイティブ部品は通常、storyboardから配置するため、viewDidLoadが実行される頃にはself.viewに追加されています。  
なので、うっかり気づかずにネイティブ部品がWKWebViewに隠れてしまっていることがあります。  
なぜネイティブボタンが表示されないのかわからなかったときは **Debug View Hierarchy** を使うと良いでしょう。  

2. 適切なConstraintsが各部品に設定されていないとき
想定に反して、各部品が配置されないことがあったとき、まさに **Debug View Hierarchy** が使えます。  
理由がConstraintsである場合、想定した画面からはみ出していたりすることが確認できるでしょう。  

いかがだったでしょうか？  
これからは筆者も積極的にDebug View Hierarchyを使っていきたいと思います。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
