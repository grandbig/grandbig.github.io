---
layout: post
title: "Swift3でUIImageを任意の角度で回転させる方法について"
date: 2017-06-11 22:43
comments: true
categories: ios swift
---

### はじめに
以前、[UIImageを任意の角度で回転させる方法について](http://grandbig.github.io/blog/2014/03/13/uiimagerotate/)を書きましたが、今回はそのSwift3版です。  
Objective-Cで書いた方法と基本的には同じではあるのですが、そのままの書き方が使えるわけではないため覚えておいて損はないはず！  

では早速見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### UIImageの回転方法
Objective-Cでは下記のように回転させていました。  

```objective-c
// 元の画像。ここではtest.pngという画像があるとします。
UIImage *image = [UIImage imageNamed:@"test.png"];

CGSize imgSize = {image.size.width, image.size.height};
// Contextを開く
UIGraphicsBeginImageContext(imgSize);
CGContextRef context = UIGraphicsGetCurrentContext();
// 回転の中心点を移動
CGContextTranslateCTM(context, image.size.width/2, image.size.height/2);
// Y軸方向を補正
CGContextScaleCTM(context, 1.0, -1.0);

// ラジアンに変換(45°回転させたい場合)
float radian = 45 * M_PI / 180;
CGContextRotateCTM(context, radian);
// 回転画像の描画
CGContextDrawImage(UIGraphicsGetCurrentContext(), CGRectMake(-image.size.width/2, -image.size.height/2, image.size.width, image.size.height), image.CGImage);

// Contextを閉じる
UIImage *rotatedImage = UIGraphicsGetImageFromCurrentImageContext();
UIGraphicsEndImageContext();

// UIImageViewに回転後の画像を設定
UIImageView *imageView = [[UIImageView alloc] init];
imageView.image = rotatedImage;
```

これをSwift3で書き直すと以下のようになります。  

```objective-c
// 元の画像。ここではtest.pngという画像があるとします。
let image = UIImage.init(named: "test")

let imgSize = CGSize.init(width: image.size.width, height: image.size.height)
// Contextを開く
UIGraphicsBeginImageContextWithOptions(imgSize, false, 0.0)
let context: CGContext = UIGraphicsGetCurrentContext()!
// 回転の中心点を移動
context.translateBy(x: image.size.width/2, y: image.size.height/2)
// Y軸方向を補正
context.scaleBy(x: 1.0, y: -1.0)

// ラジアンに変換(45°回転させたい場合)
let radian: CGFloat = 45 * CGFloat(Double.pi) / 180.0
context.rotate(by: radian)
// 回転画像の描画
context.draw(image.cgImage!, in: CGRect.init(x: -image.size.width/2, y: -image.size.height/2, width: image.size.width, height: image.size.height))

// Contextを閉じる
let rotatedImage: UIImage = UIGraphicsGetImageFromCurrentImageContext()!
UIGraphicsEndImageContext()

// UIImageViewに回転後の画像を設定
let imageView = UIImageView.init()
imageView.image = rotatedImage
```

### まとめ
さていかがでしたでしょうか？  
基本的な回転の流れは変わりませんね。  
ただ、Swift3で書いた方が心なしか自然なメソッドで書けている気がするのは筆者だけですかね...  
ということで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
