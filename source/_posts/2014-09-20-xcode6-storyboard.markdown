---
layout: post
title: "Xcode6でSingle View Applicationにチャレンジ"
date: 2014-09-20 23:18
comments: true
categories: xcode xcode6
---

###Xcode6でプロジェクトを作成しよう
さて、先日、一般ユーザ向けにXcode6のバージョンアップが解放されました。  
それに合わせて、今回はXcode6の基本的な使い方を確認していきたいと思います。  

それでは早速、Xcode6を起動してみましょう。  
![Xcode6起動後の初期ウィンドウ](/images/xcode6_1.png)  

<!--more-->

さて、 **Get started with a playground** が先頭に来ている以外はあまり気になりませんね。  
SwiftやPlaygroundの話は別の機会にということで、 **Create a new Xcode project** を選択してみましょう。  

すると、次のようなウィンドウが表示されます。  
![Xcode6でCreate a new Xcode projectを選択後に表示されるウィンドウ](/images/xcode6_2.png)  

筆者はここで、『あれ！？いつも使っていた **Empty Application** がない....』と気づきました。  
まさかと思い、あれやこれやと探し、『OtherにEmptyってのがある。これって前からあったっけ？』と思いつつEmptyでプロジェクトを作成してみると....  
![OtherにEmpty...](/images/xcode6_3.png)  
![超Empty...何もないよ...](/images/xcode6_4.png)  
TARGETもなければ、ソースも何もない。超Empty.....(泣)  

この瞬間、時代は次に進んでいるのだと諦めて、筆者はこれまで使ってこなかったSingle View Applicationに手を伸ばすことにしました。  
最大の障壁となるのが、今更ながら **Storyboardを使ったことがない** ということでした。  
これを機に覚えよう＆Xcode6でも基本操作は変わっていないよ(たぶん)と証明するために先に進み始めました。  

下記の手順でプロジェクトを作成します。  
1. Single View Applicationを選択  
2. ProductNameにStoryboardExampleと入力  
3. DevicesはiPhoneを選択  

そうして作成されたプロジェクトが下記になります。  
![StoryboardExample](/images/xcode6_5.png)  

Main.storyboardを見てみると、『何の画面サイズ？？』といったViewControllerが表示されています。  
今回はiPhoneのみに対応したアプリということで、サイズを変更したいと思います。  
サイズは下記の手順で変更ができます。  

1. 左ウィンドウでView Controllerを選択  
2. 右ウィンドウのメニューの右から3つ目の項目を選択  
3. Simulated MetricksのSizeから変更したいiPhoneのサイズを選択  

![サイズ変更方法](/images/xcode6_6.png)  

筆者の場合はせっかくなので、iPhone6のサイズである **iPhone 4.7-inch** に変更しました。  
![4.7inchのViewController](/images/xcode6_7.png)  

###StoryboardでUINavigationControllerを使おう
筆者がよく使っていたUINavigationControllerを使ってみようと思います。先ほどのView ControllerにはNavigationBarらしきものが見当たりません。これはUINavigationControllerが付与されていないためです。では、UINavigationControllerを付与しましょう。  

XcodeメニューバーのEditor > Embed in > Navigation Controllerを選択してください。  
![Navigation Controllerを追加する](/images/xcode6_8.png)  

すると、下記のようにNavigation Controllerが追加されます。(下図は見やすさのため表示倍率を変更しています。表示倍率はViewが配置されているウィンドウで右クリックするとサイズを選択可能なウィンドウが表示されます。)  
![Navigation Controllerを追加した](/images/xcode6_9.png)  
また、初めからStoryboard上に表示されていたView ControllerにNavigationBarが表示されたことがわかります。  
Navigation Controllerの大きさも先ほどと同じ手順で変えてしまいましょう。(変えなくてもコーディング上は問題ありませんが....)  

NavigationBarの良い所は画面遷移後に前の画面に戻るボタンを作らなくて良いということですね。  

続いて、Storyboard上での画面遷移の実装方法について見て行きましょう。  
画面遷移をするためには当然ながら画面が必要となるので、View Controllerを追加しましょう。また、画面遷移のアクションを起こすボタンを追加しましょう。  
![View Controllerとボタンを追加](/images/xcode6_10.png)  

そして、画面遷移の実装は、  
1. NEXTボタンを右クリックして遷移後の画面にドラッグ＆ドロップ  
2. 表示されるウィンドウから **show** を選択  
と超簡単にできます。  
その証拠に遷移後の画面にも先ほどまでなかったNavigationBarが付与されています。  

![NEXTボタンからドラッグ＆ドロップ](/images/xcode6_11.png)  
![showを選択](/images/xcode6_12.png)  
![画面遷移が実装完了](/images/xcode6_13.png)  

さて、ここまで対応が完了したら、シミュレータで表示してみましょう。  
利用するシミュレータはもちろんiPhone6です。  
![シミュレータで表示](/images/xcode6_14.png)  

上記のような表示ができましたでしょうか？  
Storyboardの使い方は案外簡単ですね！まだまだ触り程度の話ですが、これから使い込んでいこうかと思います。  

今日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
