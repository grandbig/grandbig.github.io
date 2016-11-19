---
layout: post
title: "Xcode6で様々な画面サイズに対応しよう！"
date: 2014-10-07 23:25
comments: true
categories: xcode xcode6
---

###Xcode6でAuto Layoutを使おう！
昨今はiPhone5, iPhone6, iPhone6 PlusなどiOSアプリでも様々な画面サイズに対応する必要性が出てきました。  
これまでは2画面の対応だけを考えていれば良かったのに、大きな変化ですよね？  
では、この対応をどうすれば良いのかというと、『Auto Layoutを使いましょう』ってことです。  

では早速説明していきましょう。  
StoryBoard上で右下にある**Auto Layout**ボタンをクリックします。  
![Auto Layout](/images/xcode6_autolayout1.png)  

<!--more-->

ここはXcode5と同じようなんですが、Xcode5と何やら様子が異なることに気づきます。  
それは、**左右のConstraintが0**にも関わらず、StoryBoard上では**左右に間隔があいています**。  
Xcode6で慣れれば良い話なのかもしれませんが、気になるのであれば、下記の箇所から変更できます。  

![Constraint Margin](/images/xcode6_autolayout2.png)  

左右のConstraintsを選択した状態で、上図の箇所を確認してみましょう。  
Second Itemに**Relative to margin**が紐付けられています。このチェックを外しましょう！
すると、marginがなくなるので、Constraintが0だと下図のようになります。  
![marginなし](/images/xcode6_autolayout3.png)  

ここで**Constraintを10**にするなど、あけたい間隔を設定しましょう。  

さて、ここでポイントなのが、TextFieldにはWidthの設定をつけていないことです。  
これにより、左右のConstraintが適用されるので各画面に対応できます。  

本当に対応できているのか、シミュレータや実機で確認する前にプレビューを見てみましょう。  
確認方法としては、  
1: Xcode6右上の**Show the Assistant editor**をクリック  
2: Manual, Automatic, Top Level Objects, Localization, Previewとあるので、**Preview**をクリック  

![Previewの確認方法](/images/xcode6_autolayout4.png)  

すると、4インチサイズのPreviewがデフォルトで表示されます。  
![デフォルトのPreview画面](/images/xcode6_autolayout5.png)  

4.7インチや5.5インチなどを追加する場合は、Previewの左下にある**+ボタン**をクリックして、追加したいディスプレイサイズを選択します。  
これにより、下図のように複数のディスプレイサイズでレイアウトを確認することができます。  

![複数画面](/images/xcode6_autolayout6.png)  

いかがでしたでしょうか？  
これからはiPhone6, iPhone6 Plusにも最適化したアプリを開発する必要がありますし、たいへんですが、頑張って行きたいですね！  


<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>


