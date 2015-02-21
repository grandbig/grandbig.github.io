---
layout: post
title: "UIImageを任意の角度で回転させる方法について"
date: 2014-03-13 00:01
comments: true
categories: ios
---

###UIImageを自由に回転させよう！
今日はUIImageの回転について語ります。注意して頂きたいのはUIImageViewの回転ではなく、UIImageの回転です。  
筆者は検索すればさくっと誰かが作ったものが見つかるだろうと思っていました...が、全然見つからない(汗)  
見つかっても、90°, 180°, 270°の回転といった限定されたものでした。求めているものは自由に回転させる方法なんです！！(5°とか26.8°とか...)  
しかし、粘って検索したり、いろいろと試した結果、方法がわかりました。ということで、早速説明していきましょう。  

<!--more-->

UIImageにはUIImageViewのようにtransformといったプロパティは用意されていません。回転させるためには、ガシガシソースを書く必要があります。  
下記のように...  

```objective-c
// 元の画像。ここではtest.pngという画像があるとします。
UIImage *image = [UIImage imageNamed:@"test.png"];

CGSize imgSize = {image.size.width, image.size.height};
UIGraphicsBeginImageContext(imgSize);
CGContextRef context = UIGraphicsGetCurrentContext();
CGContextTranslateCTM(context, image.size.width/2, image.size.height/2); // 回転の中心点を移動
CGContextScaleCTM(context, 1.0, -1.0); // Y軸方向を補正

float radian = 45 * M_PI / 180; // 45°回転させたい場合
CGContextRotateCTM(context, radian);
CGContextDrawImage(UIGraphicsGetCurrentContext(), CGRectMake(-image.size.width/2, -image.size.height/2, image.size.width, image.size.height), image.CGImage);

UIImage *rotatedImage = UIGraphicsGetImageFromCurrentImageContext();
UIGraphicsEndImageContext();

// UIImageViewに回転後の画像を設定
UIImageView *imageView = [[UIImageView alloc] init];
imageView.image = rotatedImage;
```

これで回転した画像を確認することができるはずです。  
因みにUIImageViewを回転させて支障がないようであれば、そっちの方が断然簡単です。  
```objective-c
UIImageView *imageView = [[UIImageView alloc] init];
imageView.image = [UIImage imageNamed:@"test.png"];

float radian = 45 * M_PI / 180; // 45°回転させたい場合
imageView.transform = CGAffineTransformMakeRotation(radian);
```

たったこれだけです。  
UIImageViewごと回転で対応できない場合の例としてはマップ上でアノテーションの画像を回転させる場合でしょう。仮にMKAnnotationViewを回転させた場合、アノテーションをタップしたときに表示されるバルーンも回転してしまうので、困ったことになります。こういったときにはUIImageを回転させるしかないんです。  

参考URL:  
[Rotate UIImage custom degree Stack Overflow](http://stackoverflow.com/questions/5102474/rotate-uiimage-custom-degree)  
[UIViewのアフィン変換](http://www.objectivec-iphone.com/animation/UIView-animation/CGAffineTransform.html)  

