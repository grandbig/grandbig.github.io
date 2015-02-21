---
layout: post
title: "iPhone専用アプリをiPadで見るとToolBarが出ない！！"
date: 2013-10-24 23:24
comments: true
categories: ios toolbar
---

###iPhone専用アプリでiPad対応
筆者はiPhone専用アプリばかりに目を向けていたので、正直iPadの対応は忘れていました。しかし、いざiPhoneアプリをiPadで起動してみると『あれっ！？ToolBarが出ない(汗)』  
これはマズイ...。格好悪い...。  
どうにか方法を探ろう！！ということでネットで調べました。これが全くもって出てこないんですよね！  
これ困っている人いないんですかね...。今日はその方法について説明します。

<!--more-->

####Xcode5での方法
さくっと説明します。ToolBarPracticeというプロジェクトを作ります。  
![DevicesはiPhone](/images/xcode_toolbar_ipad1.png)  

ここで気をつけなくてはならないのは、xibファイルの**Simulated Metrics**です。ここのSizeがデフォルト**Retina 4-inch Full Screen**になっています。このまま開発を進めるとiPadで見たときにToolBarが迷子になります。なので, ViewのDrawingの**Autoresize Subviews**のチェックを外します。この設定はパーツを置いた後でも大丈夫です。とりあえず、何も考えずに開発を進めた後、iPadで見たときにToolBarが表示されないというときでも、チェックを外すだけで対応できます。  
![Simulated Metricsの設定](/images/xcode_toolbar_ipad2.png)  

因みにiOS6とiOS7の対応は別途必要です。詳しくは[iOS6とiOS7の両方で使えるようにtoolbarを調整する方法](http://grandbig.github.io/blog/2013/09/22/xcode5-toolbar/)を参照ください。  

これで各実機でデバッグすれば、以下のようになります。  
![各実機でデバッグ](/images/xcode_toolbar_ipad3.png)  

いかがだったでしょうか？  
筆者はこれを見つけるのに苦労しましたが、知ってしまえば簡単ですね。さ〜て申請したアプリも修正しなきゃな...。
