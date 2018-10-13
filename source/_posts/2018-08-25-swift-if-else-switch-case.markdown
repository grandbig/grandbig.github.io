---
layout: post
title: "Swiftでif else VS switch"
date: 2018-08-25 20:26
comments: true
categories: ios swift
---

### はじめに
アプリ開発をしているとたまに `if xxx { ... } else if yyy { ... }` で書くか `switch`文を用いるか迷うことがあるかもしれません。  
その際には処理速度や可読性を踏まえて選択することが一般的なのではないでしょうか。  

今回は処理速度に振り切って、どちらが速いのか比較をしてみようと思います。  
( `Java` で比較している例は見られたのですが、以外にも `Swift` で比較している例が見られなかったので。)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### 比較に用いるソースコードのサンプル
まずは次のような `enum` を用意します。  

```objective-c
enum Animal {
    case dog
    case cat
    case bird
    case mouse
    case lion
}
```

この `enum` の各要素を10,000件ずつ格納されて、かつその順番をシャッフルした配列を作成します。  
以下は配列内の要素をシャッフルための処理です。  

```objective-c
// Array+Extension.swift
import Foundation

extension Array {

    mutating func shuffle() {
        for i in 0..<self.count {
            let j = Int(arc4random_uniform(UInt32(self.indices.last!)))
            if i != j {
                self.swapAt(i, j)
            }
        }
    }
}
```

続いてサンプルデータを生成する処理です。  

```objective-c
private func makeSampleData() -> [Animal] {
    var array = [Animal]()
    for _ in 0..<100000 {
        array.append(.dog)
        array.append(.cat)
        array.append(.bird)
        array.append(.mouse)
        array.append(.lion)
    }
    array.shuffle()
    return array
}
```

これを以下の `if xxx { ... } else if yyy { ... }` と `switch` の処理で比較します。  

`if xxx { ... } else if yyy { ... }` の場合は以下の通りです。  

```objective-c
// if xxx { ... } else if yyy { ... } の場合
private func calculateIfElse(animals: [Animal]) {
    let startTime = Date()
    var dog = 0
    var cat = 0
    var bird = 0
    var mouse = 0
    var lion = 0

    animals.forEach { (animal) in
        if animal == .dog {
            dog += 1
        } else if animal == .cat {
            cat += 1
        } else if animal == .bird {
            bird += 1
        } else if animal == .mouse {
            mouse += 1
        } else {
            lion += 1
        }
    }
    print("dog: \(dog), cat: \(cat), bird: \(bird), mouse: \(mouse), lion: \(lion)")
    let endTime = Date()
    print("time diff: \(endTime.timeIntervalSince(startTime))")
}
```

`switch` の場合は以下の通りです。  

```objective-c
// switch の場合
private func calculateSwitch(animals: [Animal]) {
    let startTime = Date()
    var dog = 0
    var cat = 0
    var bird = 0
    var mouse = 0
    var lion = 0

    animals.forEach { (animal) in
        switch animal {
        case .dog:
            dog += 1
        case .cat:
            cat += 1
        case .bird:
            bird += 1
        case .mouse:
            mouse += 1
        case .lion:
            lion += 1
        }
    }
    print("dog: \(dog), cat: \(cat), bird: \(bird), mouse: \(mouse), lion: \(lion)")
    let endTime = Date()
    print("time diff: \(endTime.timeIntervalSince(startTime))")
}
```

### 比較結果

5回ずつ計測した結果を比較してみます。  

`if xxx { ... } else if yyy { ... }` の計測結果  

1回目： 0.119907975196838 [sec]  
2回目： 0.112887978553772 [sec]  
3回目： 0.114282965660095 [sec]  
4回目： 0.114437937736511 [sec]  
5回目： 0.114798069000244 [sec]  

平均： 0.11526292 [sec]  

`switch` の計測結果  

1回目： 0.105566024780273[sec]  
2回目： 0.106294989585876[sec]  
3回目： 0.104817032814026[sec]  
4回目： 0.105982065200806[sec]  
5回目： 0.107689023017883[sec]  

平均： 0.10606978 [sec]  

以上より `switch` を利用した方が `0.00919314 [sec]` 速いことがわかりました。   

### まとめ
結論、処理速度を見ると複数ケースの比較が必要な場面では `switch` の方が有効なようです。  
しかし、微々たる差ではあるので、可読性も考えながら最適な方法を選択すべきかと思います。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
