---
layout: post
title: "AutoLayoutの実例（１）"
date: 2018-08-11 14:15
comments: true
categories: ios swift autolayout
---

### はじめに
幾つか現場で経験してきた `AutoLayout` の実例を少しずつ書き留めておこうと思います。  
`AutoLayout` がこれだけ当たり前にiOSに使われる世の中になったものの、実例交えて書かれている記事が少ないなと感じたためです。  
(もちろん筆者の記憶に留めておきたい気持ちもあるからですが笑)  

では、早速見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### UITableViewCell内に長さの異なるUILabelを持ったUIStackViewがある場合
下図のようなレイアウトを実現する場合  

![UITableViewCell内に長さの異なるUILabelを持ったUIStackViewがある](/images/autolayout_sample_1.png)  

これは、  

* `subTitleLabel` は文字列 `GOOD` 固定  
* `titleLabel` には可変長の文字列を表示する  
* `titleLabel` のすぐ右隣に `subTitleLabel` を表示する(要素は全て左詰め)  
* 画像は固定サイズで表示する  

というサンプルです。  

`xib` 上のUIパーツは下図の通りです。  

![xib上の構成](/images/autolayout_sample_2.png)  

#### 固定化

これは説明するまでもありませんが、 `width` や `height` に固定で `Constraint` を付与するだけです。  

まずは画像の固定化です。  

![画像の固定化](/images/autolayout_sample_3.png)  

続いて、 `subTitleLabel` の固定化です。  

![subTitleLabelの固定化](/images/autolayout_sample_4.png)  

#### 可変長のラベル

`UIStackView` 内に画像, `titleLabel` , `subTitleLabel` があります。  
画像と `subTitleLabel` は長さ指定があり、 `titleLabel` は内部の文字列が固定でないため、可変長になる必要があります。  

その場合は、 `Content Hugging Priority` と `Content Compression Resistance Priority` を利用します。  

* 画像について  
  * `Content Hugging Priority` の `Horizontal` を `251`  
  * `Content Compression Resistance Priority` の `Horizontal` を `750`  
* `titleLabel` について  
  * `Content Hugging Priority` の `Horizontal` を `250`  
  * `Content Compression Resistance Priority` の `Horizontal` を `751`  
* `subTitleLabel` について  
  * `Content Hugging Priority` の `Horizontal` を `251`  
  * `Content Compression Resistance Priority` の `Horizontal` を `750`  

このように `titleLabel` は『そのもののサイズにこだわらず』、『小さくなりにくさ』を優先させています。   

![titleLabelへのContent Hugging PriorityとContent Compression Resistance Priorityの設定](/images/autolayout_sample_5.png)  

#### 要素の左詰め

`UIStackView` 内の画像, `titleLabel` , `subTitleLabel` を左詰めで表示します。  
( `titleLabel` の文字列は可変長ですが、文字数が少なければ左に詰めたいという指示があると過程しています。 )  

その場合は `UIStackView` に `Constraint` を付与します。  
「左詰め = 右端からの `Constraint` は最低数値『以上』」と読み替えます。  

![UIStackViewの右端に以上制約を付与](/images/autolayout_sample_6.png)  

以上の3つの対応をすることで、以下のようなレイアウトを実現することができます。  

![UITableViewCell内に長さの異なるUILabelを持ったUIStackViewがある](/images/autolayout_sample_1.png)  

### まとめ
さて、本記事は実例の初回ということで、シンプル＆簡単なものを採用してみました。  
ですが、 `AutoLayout` は `View` を構造的に捉え、デフォルト値を持った優先度 ( `Priority` )の意味を深く理解する必要があります。  
深く理解した上で意図的に `Priority` を決めないと作りたい `View` を作ることはできません。  

`xib` で `GUI` 形式でペタペタ貼るからこその難しさは存在するのです。  

ということで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
