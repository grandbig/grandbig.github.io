---
layout: post
title: "LLDBの便利なp/po/ｖコマンドを覚えよう！"
date: 2019-09-07 20:02
comments: true
categories: ios swift lldb
---

### はじめに
筆者はこれまでLLDBコマンドでは `po` を利用することが多かったのですが、  
WWDC2019のビデオである[LLDB: Beyond "po"](https://developer.apple.com/videos/play/wwdc2019/429/)を視聴して改めて `v` コマンドの使いやすさを勉強しました。  

今回はその `v` コマンドについて実例を交えながら見ていきたいと思います。
( `p` や `po` コマンドでなぜか変数の中身が見れない...と思っていた方、必見です。 )  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### p/po/vコマンドについて
まず、各種コマンドについて簡単に見ていきます。   

#### poコマンドについて
`po` は `print object` の略だそうです。   
`po` は `expression` コマンドのエイリアス(ショートカット機能)であり、簡略化して書くことができます。  

具体的には、以下の通りです。  

```objective-c
(lldb) expression --object-description -- engineer

(lldb) po engineer
```

また、 `po` は式を評価した結果のオブジェクトを返すため、非常に見やすい形式で出力されます。  
早速例を見てみましょう。  

下記のような構造体が定義されているとします。  

```objective-c
struct iOSEngineer {
  var name: string
  var age: Int
  var specialty: [String]
  var swiftLevel: Int
  var obcLevel: Int
}
```

これを初期化した直後にブレークポイントで止めているとしましょう。  

```objective-c
let engineer = iOSEngineer(name: "Ichiro", age: 33, specialty: ["gps", "bluetooth", "animation"], swiftLevel: 3, obcLevel: 2)
```

ブレークポイントで止めた状態で `po` コマンドを打ち出すと、  

```objective-c
(lldb) po engineer
▿ iOSEngineer
  - name : "Ichiro"
  - age : 33
  ▿ specialty : 3 elements
    - 0 : "gps"
    - 1 : "bluetooth"
    - 2 : "animation"
  - swiftLevel : 3
  - obcLevel : 2
```

という出力結果が得られます。  

#### pコマンドについて
`p` は `print` の略だそうで、`po` と同じくコンパイルして式を評価します。  
`po` と異なるのは、出力結果の形式です。  
以下はその例です。  

```objective-c
(lldb) p engineer
(Sample.iOSEngineer) $R4 = {
  name = "Ichiro"
  age = 33
  specialty = 3 values {
    [0] = "gps"
    [1] = "bluetooth"
    [2] = "animation"
  }
  swiftLevel = 3
  obcLevel = 2
}
```

`po` と違って `$R4` のような名前がつけられていることがわかります。  
この `$R4` のような名前を引数に指定して `po` や `p` コマンドを実行することができます。  

```objective-c
(lldb) p $R4.name
(String) $R10 = "Ichiro"

(lldb) po $R4.name
"Ichiro"
```

このように以前の結果から更に値を取得するような場面で使い勝手が非常に良いです。  

もちろん `p` も `expression` コマンドのエイリアスになっています。  

```objective-c
(lldb) expression -- engineer
(MeasurementSample.iOSEngineer) $R16 = {
  name = "Ichiro"
  age = 33
  specialty = 3 values {
    [0] = "gps"
    [1] = "bluetooth"
    [2] = "animation"
  }
  swiftLevel = 3
  obcLevel = 2
}
```

#### vコマンドについて
`v` は `variable` の略のようです。  
出力形式自体は `p` コマンドと同じになっています。  

```objective-c
(lldb) v engineer
(MeasurementSample.iOSEngineer) engineer = {
  name = "Ichiro"
  age = 33
  specialty = 3 values {
    [0] = "gps"
    [1] = "bluetooth"
    [2] = "animation"
  }
  swiftLevel = 3
  obcLevel = 2
}
```

`po` や `p` と同じくエイリアスなのですが、 `expression` ではなく `frame variable` コマンドのエイリアスです。  

```objective-c
(lldb) frame variable engineer
(MeasurementSample.iOSEngineer) engineer = {
  name = "Ichiro"
  age = 33
  specialty = 3 values {
    [0] = "gps"
    [1] = "bluetooth"
    [2] = "animation"
  }
  swiftLevel = 3
  obcLevel = 2
}
```

一見、 `p` コマンドを使えば良いのでは？と思われそうですが、 `v` コマンドは以下の特徴があります。  

* コンパイルしないため、結果の出力が速い  
* メモリから変数を読み込み、動的解決を複数回繰り返すことで、サブフィールドの読み込みが可能  

上記のどちらにも言えることですが、代わりに式の評価ができないので使い分けが必要です。  

### vコマンドの便利機能
もう少し具体的に `v` コマンドの良さを見ていきましょう。  

次のように定義された構造体があったとします。  

```objective-c
protocol Engineer {
    var name: String { get set }
    var age: Int { get set }
    var specialty: [String] { get set }
}

struct iOSEngineer: Engineer {
    var name: String
    var age: Int
    var specialty: [String]
    var swiftLevel: Int
    var obcLevel:Int
}
```

上記の通り、 `iOSEngineer` はプロトコル `Engineer` に準拠しています。  
加えて独自に `swiftLevel` と `obcLevel` という変数を持ちます。  

この構造体を下記のように初期化している場面があったとします。  

```objective-c
let engineer: Engineer = iOSEngineer(name: "Ichiro", age: 33, specialty: ["gps", "bluetooth", "animation"], swiftLevel: 3, obcLevel: 2)
```

これを `p` や `po` コマンドで `iOSEngineer` 独自のフィールドである `swiftLevel` を出力させようとすると、  

```objective-c
(lldb) p engineer.swiftLevel
error: <EXPR>:3:1: error: value of type 'Engineer' has no member 'swiftLevel'
engineer.swiftLevel
^~~~~~~~ ~~~~~~~~~~
```

このようなエラーが表示されます。  
`p` や `po` コマンドは1回しか動的解決をしないため、  
`Engineer` 型として格納された `engineer` には `swiftLevel` はメンバ変数として持ち合わせていないと見なされてしまうのです。  

もちろん、`p` や `po` コマンドは式の評価が可能なので  

```objective-c
(lldb) p (engineer as! iOSEngineer).swiftLevel
(Int) $R20 = 3
```

のようにキャストすることで中身を見ることはできます。  
ですが、少々手間ではありますよね...  
(深い階層のサブフィールドを見ようとすればするほど...)  

これが `v` コマンドでは、  

```objective-c
(lldb) v engineer.swiftLevel
(Int) engineer.swiftLevel = 3
```

とキャストせずとも結果を得ることができます。  

### filter機能
もう一つ、LLDBコマンドを利用する上で役立つのが `filter` 機能です。  
折角なので使い方を見ていきましょう。  

#### filter機能の紹介
フィールドの多い変数や、その変数と同じ型の要素を数十個持つ配列などを出力させる場合、とてもではないですが特定のフィールドを見つけることは困難です。  

そんな時には `filter` 機能を利用して、特定の型であれば、特定のフィールドのみを表示するといったことができます。  
指定の方法は簡単で、  

```objective-c
type filter add <Xcodeのプロジェクト名>.<型の名前> --child <フィールド名>
```

のようにするだけです。  
筆者の例で言えば、  

```objective-c
type filter add Sample.iOSEngineer --child name
```

のような感じです。  

これにより、  

```objective-c
(lldb) v engineer
(MeasurementSample.iOSEngineer) engineer = (name = "Ichiro")
```

のように今必要のないフィールドを省略して、結果を得ることができます。  
※便宜上、 `v` コマンドを使いましたが `p` コマンドでも同じ結果を得ることができます。  

因みに、機能利用後は忘れずに `filter` を解除しましょう。  

```objective-c
type filter delete <Xcodeプロジェクト名>.<型の名前>
```

で解除できます。  

#### filter機能の限界
因みに、 `filter` 機能も万能ではなく、場合によっては複数回 `filter` をセットすることで結果を得なければいけないこともあります。  
例えば、  

```objective-c
protocol Engineer {
    var name: String { get set }
    var age: Int { get set }
    var specialty: [String] { get set }
}

struct iOSEngineer: Engineer {
    var name: String
    var age: Int
    var specialty: [String]
    var swiftLevel: Int
    var obcLevel:Int
}

struct WebEngineer: Engineer {
    var name: String
    var age: Int
    var specialty: [String]
    var htmlLevel: Int
    var cssLevel: Int
    var jsLevel: Int
}
```

のように新たに `Engineer` プロトコルに準拠した `WebEngineer` を定義し、以下のような配列を作成します。  

```objective-c
let engineer1 = WebEngineer(name: "Taro", age: 30, specialty: ["react"], htmlLevel: 2, cssLevel: 1, jsLevel: 1)
let engineer2 = WebEngineer(name: "Jiro", age: 35, specialty: ["vue"], htmlLevel: 3, cssLevel: 2, jsLevel: 4)
let engineer3 = iOSEngineer(name: "Saburo", age: 25, specialty: ["gps", "bluetooth"], swiftLevel: 3, obcLevel: 2)
let engineer4 = WebEngineer(name: "Shiro", age: 28, specialty: ["react", "vue", "angular"], htmlLevel: 4, cssLevel: 4, jsLevel: 5)

let engineers = [engineer1, engineer2, engineer3, engineer4]
```

`engineers` をそのまま `v` コマンドで出力すると少し見えにくそうなので `filter` 機能が利用したいのですが、  
`Engineer` に準拠しているからと言って  

```objective-c
(lldb) type filter add Sample.Engineer --child name
```

としても思ったようには動かず、全フィールドが出力されてしまいます。  

そこで、  

```objective-c
(lldb) type filter add Sample.iOSEngineer --child name
(lldb) type filter add Sample.WebEngineer --child name
```

としてやれば、  

```objective-c
(lldb) v engineers
([Engineer]) engineers = 4 values {
  [0] = (name = "Taro")
  [1] = (name = "Jiro")
  [2] = (name = "Saburo")
  [3] = (name = "Shiro")
```

このように期待した結果を得ることができます。  

### まとめ
さて如何でしたでしょうか。  
`p` , `po` , `v` コマンドは各々用途がありそうですが、特に `v` コマンドの利便性を感じてもらえたら幸いです。  
筆者も積極的に `v` コマンドを使っていきたいと思います。  

といったところで本日はここまで。  

参考URL:  

* [LLDB: Beyond "po"](https://developer.apple.com/videos/play/wwdc2019/429/)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
