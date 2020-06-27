---
layout: post
title: "jpegDataメソッドによるJPEG画像圧縮テスト"
date: 2020-05-30 11:30
comments: true
categories: ios swift
---

### はじめに
最近は、iPhone端末も昔に比べて飛躍的にカメラの性能も向上し、  
高品質な写真を撮ることができます。  
一消費者としては喜ばしい限りなのですが、エンジニアとしては喜んでばかりもいられません。  

というのも、画像をクラウド上で保存する機能があった場合に、  
容量と品質を天秤にかけざるを得ないことも、まだまだあるからです。  

そこで今回は、画像をJPEG圧縮することで、どの程度の品質で容量が変わるのか実験してみたいと思います。  
利用するメソッドは下記になります。  

```objective-c
func jpegData(compressionQuality: CGFloat) -> Data?
```

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### 実験
早速、実験方法ですが、  
`jpegData` の引数である `compressionQuality` に0.1刻みで0.0〜1.0の間を指定して、  
結果容量を比較してみました。    

#### 実験①：イラスト画像
まずは、区切り線の明瞭なイラスト画像を用いて実験をしてみました。  

![実験結果①](/images/jpeg_compression_sample1_result.png)  

1.0から0.9が極端に下がっていることがわかります。  
また、0.1と0.0とでは画像容量に差分がない結果となりました。  

実際に、画像を比較してみると、  

**compressionQuality: 1.0 の画像**  
![compressionQualityに1.0を指定したイラスト画像](/images/jpeg_compression_sample1_quality_1.JPG)

**compressionQuality: 0.5 の画像**  
![compressionQualityに0.5を指定したイラスト画像](/images/jpeg_compression_sample1_quality_0_5.JPG)

**compressionQuality: 0.0 の画像**  
![compressionQualityに0.0を指定したイラスト画像](/images/jpeg_compression_sample1_quality_0.JPG)

のようになります。  
当然ですが、1.0の方が全体的に滑らかで、0.0はかなり品質が落ちていることがわかります。  

※画像は[イラストレイン](https://illustrain.com/?p=9259)からお借りしました。  

#### 実験②：写真

続いて写真で実験してみました。  

![実験結果②](/images/jpeg_compression_sample2_result.png)    

イラストと同じく、1.0から0.9が極端に容量が下がっている一方で、  
0.1から0.0は同じ容量となりました。  

こちらも画像を比較してみると、  

**compressionQuality: 1.0 の画像**  
![compressionQualityに1.0を指定したイラスト画像](/images/jpeg_compression_sample2_quality_1.JPG)

**compressionQuality: 0.5 の画像**  
![compressionQualityに0.5を指定したイラスト画像](/images/jpeg_compression_sample2_quality_0_5.JPG)

**compressionQuality: 0.0 の画像**  
![compressionQualityに0.0を指定したイラスト画像](/images/jpeg_compression_sample2_quality_0.JPG)

のようになります。
やはり0.0にまでなると品質劣化が肉眼でもわかりますね。

※画像は、[ぱくたそ](https://www.pakutaso.com/20160121025post-6686.html)からお借りしました。  

### 因みに
JPEG圧縮のアルゴリズムは当然ながら、Swiftのメソッドを利用したからといって変わるわけではありません。  
なので、Mac上で画像を書き出す際に品質を指定することで確認できます笑  

![Mac上で画像をJPEG圧縮](/images/jpeg_compression_mac.png)  

JPEGのアルゴリズムに関しては、  

* [JPEGの仕組み](http://funini.com/kei/math/jpeg.shtml)  
* [JPEG画像形式の概要(圧縮アルゴリズム)。](https://www.marguerite.jp/Nihongo/Labo/Image/JPEG.html)  

などが参考になるかと思います。  

### まとめ
以上の実験からわかる通り、圧縮具合を変えることで容量に大きな変化が生まれますし、  
当然ですが、それに伴い品質も劣化していきます。  

サービス上で許容となる品質レベルは異なるでしょうし、慎重な意思決定が必要となることでしょう。  
個人的には `0.5` でも全然気にならなかったりしますが、許容範囲広すぎでしょうか。。。笑

と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
