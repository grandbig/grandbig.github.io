---
layout: post
title: "iOS13で密かに追加されたUserDefaultsへの保存制限"
date: 2019-10-19 16:49
comments: true
categories: ios ios13 userdefaults
---

### はじめに
先日ふと調べ物をしていたところ、  
[iOS 13 - Attempting to store >= 4194304 bytes of data in CFPreferences/NSUserDefaults on this platform is invalid](https://forums.developer.apple.com/thread/121527)
という何やら気になる話を見つけました。  

どうも `iOS13` からは、 `UserDefaults` に保存できる容量が `4194304 bytes` と制限が追加されたようだと言うのです。  
これは実際にやってみるっきゃない！ということで実験をしてみました。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### 実験
実際に筆者が実行した実験内容は下記の通りです。  

* `iOS13.1.3` の端末を用いて、 `UserDefaults` への保存の合計容量が `4194304 bytes` を超過すると怒られるのか  

それでは実際に見ていきましょう。  

#### 実験に利用したソースコード
制限が `4194304 bytes` であるため、文字列を保存するのは手間がかかりそうだったので、  
`Assets.xcassets` に予め用意した画像を `Data` 型に変換後、 `UserDefaults` に保存することにしました。  

用意した画像は下記になります。  

* サイズが `262KB` の画像  

実験に利用したソースコードは下記になります。  

```objective-c
// ViewController.swift
import UIKit

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
    }

    @IBAction func saveImage(_ sender: Any) {
        let image = UIImage(named: "test1")
        if let data = image?.pngData() {
            let keyName = "test\(Int.random(in: 1 ... 100))"
            print("\(data) , keyName: \(keyName)")
            UserDefaults.standard.set(data, forKey: keyName)
            UserDefaults.standard.synchronize()
        }
    }
}
```

#### 実験結果
早速、実験結果について見てみましょう。  

1回の保存サイズが `350129 bytes` だったため、12回で `4201548 bytes` となり、  
宣言である `4194304 bytes` を超えることになります。  

今回の実験では、 `print` 文で『ランダムなキー名とバイト数』を出力するように組み込み、  
端末の該当アプリに対して `Download Container...` から `UserDefaults` の中身を取得し比較してみました。  

出力結果は、  

```objective-c
350129 bytes , keyName: test59
350129 bytes , keyName: test50
350129 bytes , keyName: test51
350129 bytes , keyName: test7
350129 bytes , keyName: test58
350129 bytes , keyName: test7
350129 bytes , keyName: test91
350129 bytes , keyName: test83
350129 bytes , keyName: test54
350129 bytes , keyName: test90
350129 bytes , keyName: test80
350129 bytes , keyName: test71
350129 bytes , keyName: test25
2019-10-19 16:47:19.645111+0900 AppSample[1048:193781] [User Defaults] CFPrefsPlistSource<0x2821a5600> (Domain: com.xxx.AppSample, User: kCFPreferencesCurrentUser, ByHost: No, Container: (null), Contents Need Refresh: Yes): Attempting to store >= 4194304 bytes of data in CFPreferences/NSUserDefaults on this platform is invalid. This is a bug in NotificationSample or a library it uses
2019-10-19 16:47:19.645273+0900 AppSample[1048:193781] [User Defaults] CFPrefsPlistSource<0x2821a5600> (Domain: com.xxx.AppSample, User: kCFPreferencesCurrentUser, ByHost: No, Container: (null), Contents Need Refresh: No): Transitioning into direct mode
350129 bytes , keyName: test21
350129 bytes , keyName: test32
```

となり、 **13個目** の保存時にエラーが出力されました。
(途中キー名が被っており、上書きとなるため、その回数はスキップしています。)  

`Download Container...` で取得した `UserDefaults` の中身はどうなっているかと言うと、  

![UserDefaultsに保存された内容](/images/ios13-userdefaults.png)   

となっており、12個目までは保存されているようでした。  

厳密には少々超える分には、許容されているように見えましたが、少なくとも、はっきりと超えた場合は保存されないことがわかりました。  

### まとめ
さて如何でしたでしょうか？  
`UserDefaults` は大量の重要データを保存するのには不向きであることは、これまで通り自明であったと思いますが、  
それがより厳密化されたと言えるのではないでしょうか。  

今後に備えて、データの保存方法を今一度見直すにはいい機会かなと思います。  
と言ったところで本日はここまで。  

参考URL：  

* [Apple Developer Forums: iOS 13 - Attempting to store >= 4194304 bytes of data in CFPreferences/NSUserDefaults on this platform is invalid](https://forums.developer.apple.com/thread/121527)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
