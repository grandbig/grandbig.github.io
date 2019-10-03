---
layout: post
title: "iOSアプリのパフォーマンス計測に『os.signpost』を活用しよう！"
date: 2019-09-07 16:22
comments: true
categories: ios swift
---

### はじめに
これまでメソッドの処理時間を計測するには、  
`Date` 関数の `timeIntervalSince` メソッドを以下のように利用する場面が度々ありました。  

```objective-c
let beginDate = Date()
sample()
let endDate = Date()
let time = endDate.timeIntervalSince(beginDate)
print(time)
```

しかし実は `iOS12` から `os.signpost` を利用してもっと便利に処理時間を計測することができるようになりました。  
本日は `os.signpost` を利用したことをメモとして記録しておきます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### os.signpostのメリット・デメリット
`os.signpost` を利用すると何が良いかと言うと、  

* 最小/最大/平均の処理時間が自動で計測できる  
* CPU使用率など `Instruments` による他の計測結果と並行して状況を把握できる  

などが上げられます。  

一方でデメリットは `os.signpost` がiOS12以降でしか利用できないということです。  
よって、iOS11以下をサポートするアプリの場合、  

```objective-c
if #available(iOS 12, *) {
    ...
}
```

の条件分岐をした上で `os.signpost` を利用する必要があります。  
しかし、これは時間が解決してくれるでしょう。  
(あまり気にするデメリットではないということです。)  

### os.signpostの使い方
それでは早速、 `os.signpost` を使ってみましょう。  
`os.signpost` は特にSDKを追加することなく、 `import` が可能です。  

① `import os.signpost` を追加する  

② OSLogオブジェクトを初期化する  

[Apple Developer Document: OSLog - init](https://developer.apple.com/documentation/os/oslog/2320726-init)に記載のある通り、初期化をします。  

```objective-c
let osLog = OSLog(subsystem: "com.takahiro.MeasurementSample", category: "loop")
```

`subsystem` には `com.your_company.your_subsystem_name` の形で定義するようにと例として書かれています。  
`category` はログのカテゴリ分けに利用できるため、カテゴリ種別で命名すると良いでしょう。  

③ 計測開始時に `OSSignpostType` に `.begin` を指定した `os_signpost` メソッドを呼び出す  

[Apple Developer Document: os_signpost](https://developer.apple.com/documentation/os/3019241-os_signpost)に記載のあるメソッドを呼び出します。  

```objective-c
os_signpost(.begin, log: osLog, name: "sample")
```  

④ 計測完了時に `OSSignpostType` に `.end` を指定した `os_signpost` メソッドを呼び出す  

③と同じメソッドですが、計測完了時には `OSSignpostType` に `end` を指定します。  

```objective-c
os_signpost(.end, log: osLog, name: "sample")
```

これで、後は `Instruments` を起動して、`os_signpost` を追加して計測すればOKです。  

今回は、下記のようなサンプルを用意して計測してみます。  

```objective-c
// ViewController.swift
import UIKit
import os.signpost

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }

    @IBAction func tapButton(_ sender: Any) {
        let osLog = OSLog(subsystem: "com.takahiro.MeasurementSample", category: "loop")
        os_signpost(.begin, log: osLog, name: "sample")
        sample()
        os_signpost(.end, log: osLog, name: "sample")
    }

    private func sample() {
        for i in 0 ..< 100000 {
            print(i)
        }
    }
}
```

### os.signpostでの計測結果
`Instruments` で計測した結果は下記のように見ることができます。  

![Instrumentsのos.signpostの計測結果](/images/os-signpost.png)  

上記結果から、計測処理が `平均88.97 [ms]` かかっていることがわかります。  
またこの処理の実行中はCPUが100%に張り付いていることも見て取れます。  

今回はサンプルなので `print` で1万回出力するメソッドを計測しました。  
本物のアプリではボトルネックとなる処理を見つけ、如何にして処理を改善できないか検討することになるでしょう。  

### まとめ
いかがでしたでしょうか。  
iOS13のリリースが近づいてきた(iOS12の普及が十分に達しつつある)今だからこそ改めて `signpost` を積極的に利用していきましょう。  
といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
