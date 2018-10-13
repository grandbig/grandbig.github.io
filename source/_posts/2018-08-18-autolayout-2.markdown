---
layout: post
title: "AutoLayoutの実例（２）"
date: 2018-08-18 13:41
comments: true
categories: ios swift autolayout
---

### はじめに
前回、[AutoLayoutの実例（１）](https://grandbig.github.io/blog/2018/08/11/autolayout-1/)にて、  
『 `UITableViewCell` 内に長さの異なる `UILabel` を持った `UIStackView` がある場合』の `AutoLayout` について説明しました。  

今回は『 `UITableViewCell` 内に条件次第で `isHidden` が `true` になるパーツを持つ `UIStackView` がある場合』  
の `AutoLayout` について説明したいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### `UITableViewCell` 内に条件次第で `isHidden` が `true` になるパーツを持つ `UIStackView` がある場合
下図のようなレイアウトを実現する場合  

![UITableViewCell内に条件次第でisHiddenがtrueになるパーツを持つUIStackViewがある](/images/autolayout_sample_2_1.png)  

これは前回のサンプルの、  

* `subTitleLabel` は文字列 `GOOD` 固定  
* `titleLabel` には可変長の文字列を表示する  
* `titleLabel` のすぐ右隣に `subTitleLabel` を表示する(要素は全て左詰め)  
* 画像は固定サイズで表示する  

に加えて、以下仕様をプラスしました。  

* `hiddenLabel1` (条件次第で非表示になるオレンジ色のラベル)が `titleLabel` 等の下に配置  
* `hiddenLabel2` (条件次第で非表示になる茶色のラベル)が更にその下に配置  
* `button` が更にその下に配置  
* 条件次第で `hiddenLabel2` と `button` がセットで非表示になる  
* 条件次第で `hiddenLabel1` と `hiddenLabel2` , `button` が非表示になる  

#### レイアウトの構成
`xib` 上のUIパーツは下図の通りです。  

![xib上の構成](/images/autolayout_sample_2_2.png)  

構成について説明します。  

* 画像、 `titleLabel` , `subTitleLabel` は前回に引き続き1つの `UIStackView` 内に要素を配置しています  
  * これは3つの要素の横の関係性が場合によって変化する仕様に対応するためです。  

![画像, titleLabel, subTitleLabelを囲むUIStackView](/images/autolayout_sample_2_7.png)  

* `hiddenLabel2` と `button` は条件次第で同時に非表示になるため、`UIStackView`で囲みます  

![hiddenLabel2とbuttonを囲むUIStackView](/images/autolayout_sample_2_4.png)  

* 条件次第で `hiddenLabel1` , `hiddenLabel2` , `button` が一気に非表示になるため、さらに `UIStackView` で囲みます   

![hiddenLabel1, hiddenLabel2, buttonを囲むUIStackView](/images/autolayout_sample_2_5.png)  

* 非表示になったときに、Cellの高さが動的に変わるように、全要素を `UIStackView`で囲みます   

![全要素を囲むUIStackView](/images/autolayout_sample_2_6.png)  

#### Cellの高さを要素の表示/非表示状態次第で可変にする
`ContentView` と全要素を囲む `UIStackView`の間に `Constraint` を付与するだけです。  

![ContentViewと全要素を囲むUIStackViewの間にConstraintを付与する](/images/autolayout_sample_2_3.png)  

以前、[iOS10とiOS11で比較するUIStackViewのhiddenとConstraintエラー](http://grandbig.github.io/blog/2017/12/10/ios11-stackview-hidden/)で説明しましたが、
iOS11からは `UIStackView` を `isHidden = true` にしても、`Constraint` エラーは出なくなったので、  かなり扱いやすくなっています。  

### まとめ
さて、今回の実例は前回の実例の追加版といった位置づけで書いてみました。  
`UITableViewCell` で、条件により表示/非表示を切り替えて、動的に高さを変更する場面は多いと思います。  
今は `AutoLayout` さえ正しく使えば簡単に対応できる内容となっていますので、必ず覚えておきましょう。  

次回はもう少し複雑な `AutoLayout` の実例を紹介できればと思います。  
ということで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
