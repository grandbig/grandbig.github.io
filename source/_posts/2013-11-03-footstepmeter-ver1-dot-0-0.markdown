---
layout: post
title: "足跡計 ver1.0.0の紹介"
date: 2013-11-03 19:19
comments: true
categories: ios MyApp
---

###iPhoneアプリ『足跡計 ver 1.0.0』の紹介
さて、筆者が先日リリースしたiPhoneアプリについて紹介したいと思います。実はiPhoneアプリを申請するときに**Support URL**の入力が必須だったのです。ひとまず、筆者のブログのトップページを書いたものの、後々きちんと専用ページを作っておきたいなと思っていました。しかし、思わぬリジェクト内容から使い方の動画を作成してしまったので、何を書くか悩みました。  
そこで、アプリの操作方法ではなく**役立つ使用方法**を記載することでインストールした方に(いや、インストールされていない方にも)作り手の意図をできるだけ伝えようと決めました。  
iPhoneでの位置情報取得に興味のある方はぜひ御覧ください。  

興味がある方は下記ボタンからインストールページヘ！

<a href="https://itunes.apple.com/jp/app/zu-ji-ji/id725412788?mt=8&uo=4" target="itunes_store"style="display:inline-block;overflow:hidden;background:url(https://linkmaker.itunes.apple.com/htmlResources/assets/ja_jp//images/web/linkmaker/badge_appstore-lrg.png) no-repeat;width:135px;height:40px;@media only screen{background-image:url(https://linkmaker.itunes.apple.com/htmlResources/assets/ja_jp//images/web/linkmaker/badge_appstore-lrg.svg);}"></a>
<!--more-->

####足跡計アプリについて
基本操作は前回のブログにも記載しましたが、[You Tubeにアップした動画](http://www.youtube.com/watch?v=r2nHW-imRv4)を見て頂ければ大体わかると思います。  
ここでは本アプリの役立つ使い方について説明したいと思います。  

早速、下記をご覧ください。  
![精度の選択](/images/footStepMeter1.png)  
これはアプリで足跡計測をするときに精度を選択する画面です。全6種類の中から精度を選択することがあります。位置情報をiPhoneで扱ったことがある方ならすぐにわかると思うのですが、各選択肢はdesireAccuracyの設定を選ぶことと同義です。  
つまり  
・最高精度： kCLLocationAccuracyBestForNavigation  
・高精度： kCLLocationAccuracyBest  
・10m誤差： kCLLocationAccuracyNearestTenMeters  
・100m誤差： kCLLocationAccuracyHundredMeters  
・1km誤差： kCLLocationAccuracyKilometer  
・3km誤差： kCLLocationAccuracyThreeKilometers  
のようになっているということです。  

取得する位置情報の精度は電池消耗とのトレードオフになるので、iPhoneアプリで位置情報を取得する場合、どの程度の精度にすべきか悩むと思います。  
ここの判断を適正に行わないと電池の消耗の激しさばかりが目立ってしまい、アプリのレビューで凄まじく叩かれることになるかもしれません...。  
とは言え、実際にどの精度を選択すればどの程度の情報を取得できるのか、やってみないとわかりませんよね。そんなときに本アプリは非常に役に立つのです。  

下記をご覧ください。  
本アプリでは記録している情報の詳細を**計測中であっても**確認することができます。  
![情報の詳細を確認](/images/footStepMeter2.png)  

これを見れば、計測中に現在、選択している精度がアプリとして適正かどうか確認することができますし、なんなら計測途中で精度を再選定してテストし直すこともできます。  

また、記録した情報を解析したいということもあるでしょう。  
例えば、6種類の精度を比較するために山手線を6周して、**取得データ数, 精度のバラつき**をグラフ化したいとかですね。そんなときは本アプリの足跡履歴からデータをメールでPCに送信してしまいましょう！受け取った後、どう解析するかは完全に自由です。  
![データをメールで送信](/images/footStepMeter3.png)  

因みに、本アプリでは**標準位置情報サービス**を使用しています。大幅変更位置情報サービスは使っていません。(大幅変更位置情報サービスは今回の目的からは外れるためです。あくまでも設定可能な精度の中での比較をしてみたかったので...)  

####今後について
いかがだったでしょうか？驚くほどシンプルな内容ではなかったでしょうか？(筆者もたくさん書くことを想定していたのですが、案外書くことがなかった...)  

今後は、まだまだ現状のアプリで不便なところも多々あるので、バージョンアップに取り組んでいきたいと思っています。もっと便利にオリジナリティを出していきたいという構想はあります。  
もし、iPhoneでの位置情報の取得精度について悩まれている方がいましたら、本アプリを参考にして頂けると幸いです。なお、本アプリには全く広告はついていない&無料アプリですので、筆者にはお金は一銭も入りません笑。  

