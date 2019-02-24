---
layout: post
title: "参照渡し(inout)と『Exclusive Access to Memory』"
date: 2019-02-16 20:18
comments: true
categories: ios swift
---

### はじめに
今回は `Exclusive Access to Memory` のクラッシュについてメモ書きしておきたいと思います。  

筆者がこのクラッシュにあったのは、不用意にメソッドの引数を『参照型』にしてしまったためでした。  
本記事では、 **参照渡し** と **値渡し** を説明しつつ、 `Exclusive Access to Memory` について見ていきたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### 値渡し、参照渡しとは
まず、値渡しと参照渡しについて説明します。  

* メソッドの引数を **値型** にした場合  
  * 引数に渡されるのはコピーされた値です  
  * メソッド内でその値を変更しても、元の値には影響ありません(コピーされているので)  
* メソッドの引数を **参照型** にした場合  
  * 引数に渡されるのは、値のメモリ上のアドレスです  
  * メソッド内でその値を変更すると、元の値にも影響があります  

### Swiftでの値渡し、参照渡し
`Swift` では特に何も手を加えなければ、メソッドの引数は **値渡し** として処理されます。  
これを **参照渡し** にしたい場合は引数の型の前に `inout` をつける必要があります。  

つまり、  

```objective-c
/// 値渡しの場合
func hoge(array: [Int]) {
    ...
}

/// 参照渡しの場合
func fuga(array: inout [Int]) {
    ...
}
```

ということです。  

### Exclusive Access to Memoryとは
このエラーメッセージは、  
『同じメモリに同時にアクセスすることを禁じる』  
ことを指し示しています。  

**参照渡し** は値のメモリ上のアドレスにアクセスするため、  
**参照渡し** の場合に気をつけなければなりません。  

因みに、これはSwift4で取り入れられた制御とのことです。  
詳しくは下記参照のこと...  
[0176-enforce-exclusive-access-to-memory.md](https://github.com/apple/swift-evolution/blob/master/proposals/0176-enforce-exclusive-access-to-memory.md)

### Exclusive Access to Memoryに遭遇した場合の解決方法
では、どうやって解決すれば良いのでしょうか。  
考えられる手法としては下記2つかなと思っています。  

* メモリへのアクセスタイミングをズラす  
* **参照渡し** から **値渡し** に変更する  

前者で解決するのであればそれに越したことはないのですが、  
iOSの `Lifecycleメソッド` や `Delegateメソッド` など、こちらでタイミングをズラすことのできない場合は、 **値渡し** に変更せざるを得ないこともあるでしょう。  

### まとめ
参照渡しを利用するときは、メモリアクセスに十分注意して設計する必要がありますね。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
