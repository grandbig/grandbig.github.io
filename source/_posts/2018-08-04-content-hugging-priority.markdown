---
layout: post
title: "今更だけど正しく身につけよう『Content Hugging Priority』と『Content Compression Resistance Priority』"
date: 2018-08-04 16:42
comments: true
categories: ios swift autolayout
---

### はじめに
今回は `AutoLayout` の中でもしっかりと知っておきたい以下2つを紹介します。  

* `Content Hugging Priority`  
* `Content Compression Resistance Priority`  

上記2つを利用することで、各サイズでの想定されたデザインを再現することができます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### 題材の紹介
今回説明に利用する題材は以下とします。  

* `UILabel` 2つが配置されたカスタム `UITableViewCell` を持つ `UITableView` の表示  
* `UITableViewCell` 内に2つの `UILabel` を配置するために `UIStackView` を利用  

![説明用の題材](/images/content_hugging_priority_1.png)  

上記に表示した `UITableViewCell` に付与された制約は次の通りです。  

* `UIStackView` の左右に `SuperView` に対して `8pt` の制約を付与  
* `UIStackView` のY位置を `SuperView` の `CenterY` と一致させる制約を付与  
* `subTitleLabel` の `width` を `100pt` に指定  

![付与している制約](/images/content_hugging_priority_2.png)   

### Content Hugging Priority

先程の紹介では `subTitleLabel` に `width: 100pt` の制約を付与していました。  
もし、この制約が不要で、以下のようなデザイン指定がある場合、どのように対処すれば良いでしょうか。  

* `titleLabel` の `width` を `subTitleLabel` よりも広く取りたい  
* (逆を言えば、) `subTitleLabel` の `width` はコンテンツサイズ以上に広くしたくない  

その場合は、  
`titleLabel` の `Horizontal` の `Content Hugging Priority` を、  
`subTitleLabel` の `Horizontal` の `Content Hugging Priority` よりも低く設定します。  
※デフォルト値は `251` です。  

![Content Hugging Priorityの指定](/images/content_hugging_priority_3.png)  

この結果、  

![Content Hugging Priorityを設定した結果](/images/content_hugging_priority_4.png)  

のようになります。  
これは `subTitleLabel` のテキストである `GOOD` が全文表示される最低サイズを優先して設定していることを指しています。  
逆に、 `titleLabel` は中身のテキストに寄らず、横幅が広くなっているのも同じ理由です。   

つまり、  
`Content Hugging Priority` が高いと、**コンテンツのサイズを優先する** ことがわかります。  

### Content Compression Resistance Priority
上記で説明した際には `subTitleLabel` のテキストが `GOOD` でした。  
もしも、 `subTitleLabel` のテキストが長文だった場合、どうなるでしょうか。  

答えは、  

![subTitleLabelが長文の場合](/images/content_hugging_priority_5.png)   

です。  

`subTitleLabel` が長くなり `titleLabel` の文字が1文字しか表示されなくなってしまいました。   

ここで、テキストの表示重要度が `titleLabel` の方が `subTitleLabel` よりも高いとしましょう。  
それを実現するために `Content Compression Resistance Priority` を利用します。  

`titleLabel` の `Horizontal` の `Content Compression Resistance Priority` を、  
`subTitleLabel` の `Horizontal` の `Content Compression Resistance Priority` よりも高く設定します。  
※デフォルト値は `750` です。  

![Content Compression Resistance Priorityの指定](/images/content_hugging_priority_6.png)  

この結果、  

![Content Compression Resistance Priorityを設定した結果](/images/content_hugging_priority_7.png)  

のようになります。  
これは `titleLabel` のテキストを極力表示するように優先して設定されていることを指しています。  

つまり、 `Content Compression Resistance Priority` が高いと、  
文字通り、 **小さくなりにくさの優先度を高くしている** ということです。  

### おまけ
因みに、 `subTitleLabel` を完全に非表示にしたくないと言った要望がある場合には、  
`subTitleLabel` の `width` に最低サイズを指定すれば良いでしょう。  

![subTitleLabelにwidthを設定](/images/content_hugging_priority_8.png)  

これは以下理由により実現されます。  

* `Content Compression Resistance Priority` の `Priority` は `751`  
* `width` に付与した `Priority` は `1000`  

![subTitleLabelにwidthを指定した結果](/images/content_hugging_priority_9.png)  

### まとめ
上記をまとめます。  

* `Content Hugging Priority` が高い = コンテンツサイズを優先する  
* `Content Compression Resistance Priority` が高い = 小さくなりにくさを優先する  
* どの制約が優先して適用されるかは `Priority` の値に従って決まる  

さて如何でしたでしょうか？  
今更ながら実例を交えてきちんと整理しておきたい気持ちが強くなり、今回のブログ記事となりました。  
iOSアプリを開発する際に、 `AutoLayout` スキルをないがしろにすることはできません。  

ぜひぜひ今後とも強めていきたいところですね。  
ということで、本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
