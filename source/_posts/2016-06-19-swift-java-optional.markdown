---
layout: post
title: "SwiftとJava8でOptional型を比較してみよう！"
date: 2016-06-19 21:12
comments: true
categories: ios swift java
---

####SwiftとJava8のOptional型を比較してみよう！
本日はSwiftとJava8のOptional型について比較してみたいと思います。  
最近チラチラとJava8を見かける機会が多いのですが、Swiftと同じくOptional型という概念があるんだ〜と何となく思っていました。  
が、実際に全く同じというわけではないと思うので比較したいと思ったわけです。  
ということで早速見ていきましょう！  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

#####SwiftのOptional型とは
まずはSwiftのOptional型から見ていきましょう。  
SwiftはObjective-Cでよく発生していた『思いがけず`nil`が入ったプロパティにアクセスしてExceptionが発生する』という事象を回避できる **Optional型** という概念があります。  
Swiftでは **Optional型** と **非Optional型** を変数定義時に明示的に利用することでこういったExceptionを回避することが可能なのです。  

* Optional型: 変数に`nil`を代入することを **許容します**  
  * Optional型はデータ型の末尾に **『?』** か **『!』** をつけます  
    * 『?』: 一般的なOptional型です  
    * 『!』: 暗黙的Optional型です  
* 非Optional型: 変数に`nil`を代入することを **許容しません**  

```objective-c
// Optional型
var hoge: Int?
var fuga: Int!

// 非Optional型
var piyo
```

#####Optional型から値を取得する方法は
さて、Optional型の宣言方法は先程話した通りです。  
実際に値を取得する際はOptional型で定義された変数を **アンラップ** する必要があります。  

ここで注意したいのが、先に紹介した『!』を使った暗黙的Optional型の場合は **自動的にアンラップする** ので開発者側はアンラップさせる必要がないということです。  

アンラップ方法は下記の3通りです。  

* Forced Unwrapping: 変数に『!』をつけます  
* Optional Binding: `if`文を利用します  
* Optional Chaining: 変数に『?』をつけます  

Forced UnwrappingとOptional Bindeingの例は下記です。  

```objective-c
var hoge: Int? = 1

// Forced Unwrapping
print(hoge!)

// Optional Binding
if let fuga = hoge {
	print(fuga)
}
```

続いてOptional Chainingの例を書きます。  
まずは下記のようなクラスを定義します。  

```objective-c
class Hoge: NSObject {
	
	func hogehoge() {
		print("hogehogeメソッドを実行しました")
	}
}
```

上記で定義したクラスの`hogehoge`メソッドをOptional Chaningを利用して実行します。  

```objective-c
var hoge: Hoge? = Hoge()

// Optional Chaining
hoge?.hogehoge()
```

このようにSwiftではコーディングしていく段階でかなり`nil`に注意する必要があることがわかります。  

#####Java8のOptional型とは
次にJava8のOptional型について見ていきましょう！  
Java8ではOptionalを利用することで  

* `null`の可能性がある変数をラップしておくことで、値を安全に取り出せる  
* 実行したメソッドが`null`を返却する可能性がある場合に、場合分けを短く書ける  

というメリットがあります。  
では実際に使い方を見ていきましょう！  

```java
String hoge = "hoge";

// Optional型にラップする
Optional<String> hogeOpt = Optional.of(hoge);

// Optional型から値を取り出す
// getメソッドを使う
System.out.println(hogeOpt.get());

// orElseメソッドを使う
System.out.println(hogeOpt.orElse("default data"));

// orElseGetメソッドを使う
System.out.println(hogeOpt.orElseGet(() -> "default data"));

// ifPresentメソッドを使う
hogeOpt.ifPresent( hoge -> {
	// 値があったときにログ出力
	System.out.println(hoge);
});
```

上記を見ると幾つか値の取得方法があることがわかると思います。  
それぞれSwiftの表記に近しいところがあると感じました。  

例えば、`get`メソッドは値が`null`だった場合に`NoSuchElementException`を投げます。  
逆に言えば、確実に`null`が来ない場合の`get`メソッドが利用できると言えます。  
これはSwiftで言うところの『Forced Unwrapping』です。  

`orElse`や`orElseGet`はSwiftで言うところのnil合体演算子である『??』を利用しているのとほぼ同等です。  
(Swiftであれば`let val = hoge ?? "default data"`みたいな感じですね。)  

`ifPresent`メソッドもSwiftで言うところの『Optional Binding』と言えるでしょう。  

####まとめ
さて如何でしたでしょうか？  
Android Studio 2.2からJava8のラムダ式サポートなんて話も聞こえてきますし、  
スマートフォンのアプリ開発者は今後、SwiftにもJava8にも関わっていく可能性が十分ありえます。  
そうなったときに学習コストが高いな〜と避けるのではなく、案外触ってみると「あれ！？似てる...」なんてことがあるかもしれません。  
そんな近未来！？を夢見つつブログを書いてみたのでした。  
ということで今日はここまで。  


参考:  

* [どこよりも分かりやすいSwiftの"?"と"!"](http://qiita.com/maiki055/items/b24378a3707bd35a31a8)  
* [Java8のOptionalの使い方について](http://www.task-notes.com/entry/20150708/1436324400)  


<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
