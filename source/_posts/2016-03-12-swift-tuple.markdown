---
layout: post
title: "SwiftでTupleを活用しよう！"
date: 2016-03-12 23:09
comments: true
categories: ios swift
---

####Tupleとは
さて、本日はSwiftの **Tuple** について見ていきます。  
**Tuple** とは『複数の型の値を一組にまとめる』ときに利用します。  

続いて具体的な使い方を紹介します。  

#####valueのみの格納

```objective-c
let sample = (201603, "TEST", true)
print("0番目の値は\(sample.0), 1番目の値は\(sample.1), 2番目の値は\(sample.2)")
```

これを実行すると、  

```objective-c
0番目の値は201603, 1番目の値はTEST, 2番目の値はtrue
```

のように出力されます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

#####key, valueの格納

```objective-c
let sample = (num: 201603, str: "TEST", flag: true)
print("0番目の値は\(sample.num)", 1番目の値は\(sample.str), 2番目の値は\(sample.flag))
print("0番目の値は\(sample.0), 1番目の値は\(sample.1), 2番目の値は\(sample.2)")
```

これを実行すると、  

```objective-c
0番目の値は201603, 1番目の値はTEST, 2番目の値はtrue
0番目の値は201603, 1番目の値はTEST, 2番目の値はtrue
```
のように出力されます。  
key, valueの関係で書いても、数値指定が可能です。  

数値指定ができることから、部分的にkeyを指定することができます。  

```objective-c
let sample = (201603, str: "TEST", flag: true)
print("0番目の値は\(sampel.0), 1番目の値は\(sample.str), 2番目の値は\(sample.flag)")
```

これを実行すると、  

```objective-c
0番目の値は201603, 1番目の値はTEST, 2番目の値はtrue
```

と出力されます。  

####TupleとArray, Dictionaryの違いは？
**Tuple** が真新しく見えるかというと実はそうでもないというのが筆者の感想でありました。  
というのも、ここまでの話では『ArrayやDictionaryで代用できるのでは？』と思えたためです。  

それぞれ比較してみましょう。  

#####TupleとArrayを比較
まずは、 **Tuple** とArrayの比較です。  

```objective-c
// Array
let arraySample = [201603, "TEST", true]
print(arraySample[0])

// Tuple
let tupleSample = (201603, "TEST", true)
print(tupleSample.0)
```

* 2つの違い
    * 書式が異なる(Tupleの方が簡易)
    * Arrayは要素の追加と削除が可能だが、Tupleはできない
* 2つの同じところ
    * 異なる型の値を格納できる(Arrayは`AnyObject`にすれば良い)
    * 値渡し

#####TupleとDictionaryを比較
次に、 **Tuple** とDictionaryの比較です。  

```objective-c
// Dictionary
let dictSample = ["numValue": 201603, "strValue": "TEST", "flagValue": true]
print(dictSample["numValue"])

// Tuple
let tupleSample = (numValue: 201603, strValue: "TEST", flagValue: true)
print(tupleSample.numValue)
```

**Tuple** とDictionaryの違いと同一点はArrayと比較した場合と同じです。  

* 2つの違い
    * 書式が異なる(Tupleの方が簡易)
    * Dictionaryは要素の追加と削除が可能だが、Tupleはできない
* 2つの同じところ
    * 異なる型の値を格納できる(DictionaryはValueを`AnyObject`にすれば良い)
    * 値渡し

####まとめ
正直、『こういったところでは  **Tuple** を必ず利用したい』という利用シーンがあまり思い浮かびません...。  
書式が簡単でとっつきやすいという良さはあるかもしれませんが、  
実際に利用した際にその本当の意義を感じることができるのかもしれませんね。  

と言ったところで、本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
