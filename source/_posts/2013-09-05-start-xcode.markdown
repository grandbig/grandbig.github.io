---
layout: post
title: "Xcodeを始めてみよう！"
date: 2013-09-05 23:56
comments: true
categories: xcode
---

##Xcodeとは
アップルが提供している統合開発環境(IDE)の名称です。  
『iPhone/iPadアプリを開発したい！』という方、Mac PCを買ったら速攻インストールです！  
(XcodeはもちろんApp Storeからインストールできます）  

この記事時点での最新版は4.6.3ですが、今月中にはXcode5.0が発表されることでしょう。  
さてさて、早速Xcodeを起動してみましょう。  

###プログラミングまでの準備
Xcodeを起動すると次のウィンドウが表示されます。
![Welcome to Xcode](/images/xcode1.png)

<!--more-->

プロジェクトを新規作成する場合は『Create a new Xcode project』を選択しましょう。
すると次の画面で『Choose a template for your new project』と言われます。
![Choose Project](/images/xcode2.png) 
とは言え、何だか選択肢が多いですね…。ここで選択を失敗すると一気に時間がかかってしまいそうな予感がしますね……。  
本来は*Single View Application*の方が初心者向きかもしれないのですが、筆者はあまり慣れておりません(汗)  
ってことで今回は*Empty Application*を選択します。(この違いについてはいつの日か説明したいと思います)  

すると今度は『Choose options for your new project』と言われます。
![Choose Options](/images/xcode3.png)
入力するのは『Product Name』, 『Organization Name』,『Company Identifier』の3つです。後から変更も可能なので、ここでは『blog\_test』,『筆者の名前』,『jp.co.test』としておきましょう。  
  
はい！新規プロジェクト作成できました〜。
![Choose Options](/images/xcode4.png)
  
次回はアプリでも作ってみようかな
