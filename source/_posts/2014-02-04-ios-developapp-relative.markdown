---
layout: post
title: "iOSアプリを相対表示で作りたい！！"
date: 2014-02-04 22:52
comments: true
categories: ios
---

###iOSアプリでpixel表示を使わない？
さて、皆さんは**iPhoneとiPadに対応**、**iOS6とiOS7に対応**を仕様として求められたときにどうやって作るのが最も簡単だと思いますか？  
Storyboardを使う方法, 空プロジェクトからnib/xibを使う方法, ただひたすらにコードを書く方法など人によって様々かと思います。筆者は最近、**ただひたすらにコードを書く方法**を取っています。理由は結局, iPhone/iPadの対応やiOS6/iOS7の対応のためにそこだけコードを書いて対応したりすることが生じるためです。それならばいっそのこと、0からコードを書いた方が勉強にもなるし、気持ちが良いのです。  
しかし、周りからのは『コードで書くとデバッグしないとイメージが湧かないんだよなぁ』とか、『pixel指定で作るとか面倒くさくね〜？』なんて言われたりするんですよね(汗)  
この両方に通ずることは**相対表示で画面の構造が作れれば良いのに**なんてことなんじゃないかと勝手に思いました。  
なので、本日は開発者がpixel指定を意図せずiOSアプリの画面を作成できるような関数を定義してみたいと思います。  

<!--more-->

####関数の定義
さて、早速定義します。  
横幅や縦幅を指定したパーセンテージの長さに変換したい場合は
```
// 横幅の相対変換
- (float) relativeWidth:(CGRect)frame
				percent:(int)percent
{
	// 引数のframeから横幅を取得
	float parentWidth = frame.size.width;
	float childWidth = parentWidth * percent / 100;

	return childWidth;
}
```
で良いでしょう。  
横幅, 縦幅にとらわれたくなければ  
```
- (float) relativeLength:(float)length
				 percent:(int)percent
{
	float childLength = length * percent / 100;

	return childLength;
}
```
とすれば良いでしょう。  
横幅, 縦幅を同時に変換したければ  
```
- (CGRect) relativeFrame:(CGRect)frame
			percentWidth:(int)percentWidth
		   percentHeight:(int)percentHeight
{
	float parentWidth = frame.size.width;
	float parentHeight = frame.size.height;

	float childWidth = parentWidth * percentWidth / 100;
	float childHeight = parentHeight * percentHeight / 100;

	CGRect rect;
	rect.size.width = childWidth;
	rect.size.height = childHeight;

	return rect;
}
```
とできると思います。  

これでHTML, CSSでいうところの**position: relative**で**width: 30%; height: 20%;**のような形を取ることができます。  
動作確認してないけど、たぶん行けるでしょう...。
