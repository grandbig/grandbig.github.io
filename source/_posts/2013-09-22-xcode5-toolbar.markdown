---
layout: post
title: "iOS6とiOS7の両方で使えるようにtoolbarを調整する方法"
date: 2013-09-22 12:41
comments: true
categories: ios toolbar
---

###iOS6/iOS7で使えるようにtoolbarを調整しよう！
どうも。筆者が先週末から１週間ほど風邪で寝込んでいる間にiPhone5S/5Cが発売されたようですね。また、それに先駆けてiOS7とXcode5の正式版も一般にリリースされたということで完全に乗り遅れてしまいました(泣)  
そろそろ体力も復帰してきたということで気になっていたXcode5に触れてみることにしました。  
今回はiOS7になったことで見た目はささやかな違いなんですが、実は結構面倒かもしれないtoolbarの変換に着目しました。まずはその違いを実際に見て頂きましょう。
![twitterでtoolbarを比較しよう](/images/ios6_7_difference.png)

<!--more-->

このようにiOS7対応アプリではステータスバーを含めたtoolbarを作成する必要があります。もちろんiOS6ではステータスバーを含まないのでtoolbarの高さが異なります。  
『単純にtoolbarの高さを調整すれば良いのでは？』と思うかもしれませんが、下記画像を見て頂くとわかるようにdisabledになっていて変更できません...  
![変更できないtoolbarの高さ](/images/noChange_toolbar_height.png)  
これを変更するために、『Show the File inspector』を選択して、『Use Autolayout』のチェックを外します。そして、iOS6からの目線でiOS7を調整するために(慣れの問題ですが…)『View as』をiOS6.1 and Earlierに変更します。  
![Use Autolayoutのチェックを外す](/images/nocheck_use_autolayout.png)  
それによって『Show the Size inspector』で_iOS6/7 Deltas_が表示されるようになります。  
![iOS6/7 Deltasが表示される](/images/iOS6_7_Deltas.png)  
ステータスバーのheightは20pxなので、iOS7はiOS6よりもheightが20px高くなります。また、位置はiOS6での最上位置(0px)よりも上に20px つまり-20pxになります。  
![iOS6/7 Deltasに値を設定](/images/iOS6_7_Deltas_set.png)  

これでtoolbarの設定は完了です。因みにtoolbarと合わせてWebViewやMapViewを使うときには同じくこれらのパーツも調整する必要がありますのでご注意を。
