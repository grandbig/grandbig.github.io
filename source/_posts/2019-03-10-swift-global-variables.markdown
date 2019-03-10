---
layout: post
title: "Swiftのグローバルプロパティとスタティックプロパティは自動でlazyになる"
date: 2019-03-10 00:40
comments: true
categories: ios swift
---

### はじめに
今日はSwiftでグローバルプロパティを定義した時の挙動についてのメモです。  

Swiftではプロパティを定義する時に、わざと遅延評価させるための `lazy` という修飾子があります。  
`lazy` をつけることで、そのプロパティが利用される時に、初期化されメモリが消費されるため、メモリの効率性を上げることに一役買います。  

実際の定義は下記の通りです。  

```objective-c
// ViewController.swift

class ViewController: UIViewController {
    lazy var localProp = "local"
    ...
}
```

これがSwiftで **グローバルプロパティ** や **スタティックプロパティ** を定義した時に自動的に付与されます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### 実験
では、実際に実験してみましょう。  

#### 事前準備
まずは下記のような `Sample` クラスを定義します。  

```objective-c
// 実験用のクラス

class Sample {
  let label: String

  init(label: String) {
      self.label = label
      print("initされたよ！ - \(label)")
  }

  deinit {
      print("deinitされたよ！ - \(label)")
  }
}
```

そして、下記のような `Storyboard` を用意します。  

![Storyboard](/images/swift_global_variables_1.png)  

各種機能を箇条書きで述べると下記の通りです。  

* 中央の『`Next Page`』ボタンを持つVCが `FirstViewController`  
* 右端のVCが `SecondViewController`  
* `FirstViewController` の『`Next Page`』ボタンをタップすると `SecondViewController` に遷移する  
* `SecondViewController` に各種プロパティを定義  

```objective-c
// SecondViewController.swift
import Foundation
import UIKit

/// グローバルプロパティ
var global_prop = Sample(label: "global_prop")

class SecondViewController: UIViewController {

    /// 通常のローカルプロパティ
    var local_prop = Sample(label: "local_prop")
    /// 遅延実行されるローカルプロパティ
    lazy var local_lazy_prop = Sample(label: "local_lazy_prop")
    /// スタティックプロパティ
    static var static_prop = Sample(label: "static_prop")

    override func viewDidLoad() {
        super.viewDidLoad()

        // 各種プロパティにアクセスする
        print(global_prop)
        print(local_prop)
        print(local_lazy_prop)
        print(SecondViewController.static_prop)
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }

    deinit {
        print("SecondViewControllerがdeinitされました")
    }
}
```

#### 実験結果
準備が整ったので、実際に挙動を見ていきましょう。  

まずは、 `FirstViewController` の『`Next Page`』をタップして `SecondViewController` に遷移した場合です。  

結果は、  

```objective-c
initされたよ！ - local_prop
initされたよ！ - global_prop
initされたよ！ - local_lazy_prop
initされたよ！ - static_prop
```

の順番でログが出力されました。  

このことから、  

* `local_prop` は `SecondViewController` 初期化時に初期化される  
* `global_prop` はアクセス時に初期化される  
* `local_lazy_prop` はアクセス時に初期化される  
* `static_prop` はアクセス時に初期化される  

ことがわかります。  

続いて、 `SecondViewController` のナビゲーションバーの『`Back`』をタップして前の画面に戻ってみます。  

```objective-c
SecondViewControllerがdeinitされました
deinitされたよ！ - local_prop
deinitされたよ！ - local_lazy_prop
```

のようにログが出力されました。  

このことから、  

* `local_prop` は `SecondViewController` が `deinit` されるタイミングで `deinit` される  
* `local_lazy_prop` は `SecondViewController` が `deinit` されるタイミングで `deinit` される  
* `global_prop` は `SecondViewController` が `deinit` されても `deinit`  **されない**  
* `static_prop` は `SecondViewController` が `deinit` されても `deinit`  **されない**  

ことがわかります。  

さらに、再度 `FirstViewController` の『`Next Page`』をタップして `SecondViewController` に画面遷移してみます。  

すると、  

```objective-c
initされたよ！ - local_prop
initされたよ！ - local_lazy_prop
```

のようにログが出力されました。  

このことから、  

* `local_prop` は `SecondViewController` 初期化時に初期化される  
* `local_lazy_prop` はアクセス時に初期化される  
* `global_prop` と `static_prop` は既に初期化済みなので、初期化されない  

ことがわかります。  

### グローバルプロパティ vs スタティックプロパティ
因みに、グローバルプロパティとスタティックプロパティの挙動は下記の点で似ています。  

* 特定のクラスを初期化しなくても利用できる  
* 特定のクラスが `deinit` されてもプロパティ自体は `deinit` されない  

そうなると、グローバルプロパティとスタティックプロパティそれぞれ使い分けに疑問を抱くかもしれません。  
しかしながら、この2つは全く利用用途が異なります。  

* グローバルプロパティ  
  * グローバルにアクセスする必要がある場合に利用する  
  * 特定のクラスに依存するような関係性を持たない  
  * 名前空間はグローバル(全体)に影響があるため、十分注意が必要  
* スタティックプロパティ  
  * 特定のクラスを初期化せずともアクセスする必要がある場合に利用する  
  * 特定のクラスに依存する関係性ではある  
    * 特定のクラスを初期化せずとも利用できるが、特定のクラスが持つ役割に含まれる  
  * 名前空間は特定のクラス内に留まる  

上記から、より適切な使い分けをするように気をつけましょう。  

### まとめ
ではまとめます。  

* グローバルプロパティは自動で `lazy` 付与と同じ扱いになる  
* スタティックプロパティは自動で `lazy` 付与と同じ扱いになる  
* グローバルプロパティとスタティックプロパティは特定のクラスが `deinit` されても `deinit` されない  
* グローバルプロパティはグローバルにアクセスが必要な場合に利用する  
* スタティックプロパティは特定のクラス内に定義して利用することで、その役割を明確にして利用する  

ということで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
