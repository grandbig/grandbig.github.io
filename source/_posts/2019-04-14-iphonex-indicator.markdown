---
layout: post
title: "iPhoneX時代のNetwork Indicatorについて"
date: 2019-04-14 00:30
comments: true
categories: ios swift
---

### はじめに
これまでのiPhone8/8Plus/SEなどでは通信中に下図のような `Network Indicator` が表示されていました。  

![従来のNetwork Indicator](/images/iphone-indicator.png)  

しかしながら、iPhoneX系統では、下図のように `Network Indicator` の表示がなくなってしまいました。  

![iPhoneX系統ではNetwork Indicatorがない](/images/iphonex-indicator.png)  

今回は、この `Network Indicator` に焦点を当てたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### iPhoneX時代のNetwork Indicatorは？
『iPhoneXではステータスバーに`Network Indicator`を表示しない』方針を取ったのは、  
紛れもなくAppleなわけですから、何らかの他の方法でユーザに通信中であることを伝えていると思われます。  

そこで幾つかのApple標準アプリを見ていきたいと思います。  

まず、`App Store` を見てみると、

* 読み込まれるまでは真っ白  
* 読み込みの早い文字を先に表示し、画像系は枠だけ表示する  

ようになっていました。  
Apple標準アプリ以外でも、上記手法を取り入れているアプリは結構多そうでした。  

![アプリの読み込み例：Placeholder](/images/iphonex-placeholder.png)  

続いて、`Safari` を見てみると、  

* アドレスバー下にプログレスバーを表示する  

ようになっていました。  

![アプリの読み込み例：ProgressBar](/images/iphonex-progressbar.png)  

### iPhoneXで従来型Network Indicatorを表示するためには？
このように現在のアプリでは、従来のような密やかな`Network Indicator`よりも、  
この先にどんな画面が表示されるか想像させたり、  
あとどの程度で読み込みが終わるのか進捗を伝えたりなど、  
より具体性を伝達するように変わってきている気がします。  

しかしながら、従来のような密やかな通信状態の伝達が完全に不要になったかと言うと、それはアプリ仕様に依存するところでしょう。  
では、そんなアプリを開発するときに、どうすればよいかと言うと、  
[FTLinearActivityIndicator](https://github.com/futuretap/FTLinearActivityIndicator)を利用してみるのも一つの手かなと思います。  

#### FTLinearActivityIndicatorで密やかなNetworkIndicatorを出すためには
非常に簡単に実装ができます。  

まずは、`Indicator`の`configure`を実行します。  

```objective-c  
// AppDelegate.swift
import UIKit
import FTLinearActivityIndicator

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    ...
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.
		    UIApplication.configureLinearNetworkActivityIndicatorIfNeeded()
        return true
    }
    ...
}
```

続いて、必要なタイミングで、  
`UIApplication.shared.isNetworkActivityIndicatorVisible`を`true`にするだけです。  
※非表示にする場合は`false`にセットし直すだけです。  

#### FTLinearActivityIndicatorはどんなことをやっているのか
まず、ライブラリの構成としては、  

```objective-c
FTLinearActivityIndicator
  ├── FTLinearActivityIndicator.swift
  └── UIApplication+LinearNetworkActivityIndicator.swift
```

となっています。  
`FTLinearActivityIndicator.swift`はクラス名通り、`NetworkIndicator`の表現やアニメーション系のメソッドを提供しています。  

一方で、`UIApplication+LinearNetworkActivityIndicator.swift`は、  

* `UIApplication`の拡張  
* `UIViewController`の拡張  
* `DispatchQueue`の拡張  

と3つに分かれており、`FTLinearActivityIndicator`を制御するためのメソッドが用意されています。  

具体的に実装の中身を見ていきます。  
先程の`AppDelegate`の`didFinishLaunchingWithOptions`で実行していた`configureLinearNetworkActivityIndicatorIfNeeded()`内の処理は下記のようになっています。    

```objective-c
@objc final public class func configureLinearNetworkActivityIndicatorIfNeeded() {
    if #available(iOS 11.0, *) {
			// detect iPhone X
			if let window = shared.windows.first, window.safeAreaInsets.bottom > 0.0 {
				if UIDevice.current.userInterfaceIdiom != .pad {
					configureLinearNetworkActivityIndicator()
				}
			}
		}
}
```

これを見ると、ライブラリ側で`iPhoneX`系端末なのか/`iPad`ではない端末なのかを判別してくれていることがわかります。  
既に全画面表示の `iPad` が世に出ているため、作成者に`PullRequest`を送るチャンスかもしれないですね！  

続いて、もう少し深ぼると...  
上記メソッド内で実行している`configureLinearNetworkActivityIndicator()`の中身は、  

```objective-c
class func configureLinearNetworkActivityIndicator() {
    // 説明(1)
		DispatchQueue.once {
      // 説明(2)
			let originalSelector = #selector(setter: UIApplication.isNetworkActivityIndicatorVisible)
			let swizzledSelector = #selector(ft_setNetworkActivityIndicatorVisible(visible:))
			let originalMethod = class_getInstanceMethod(self, originalSelector)
			let swizzledMethod = class_getInstanceMethod(self, swizzledSelector)
			method_exchangeImplementations(originalMethod!, swizzledMethod!)
		}
		UIViewController.configureLinearNetworkActivityIndicator()
}
```

になっています。  

重要ポイントを説明すると、  

* 説明(1)  
  * `DispatchQueue.once`の内部実装を見ると、ファイル名/メソッド名/行数からトークンを作成し、実行回数を管理しています  
  * これにより、誤って無駄に重複実行したとしても、防いでくれるようになっているようです  
* 説明(2)  
  * ライブラリの存在を意識する必要なく機能を利用できるように`method_exchangeImplementations`を使っています 　
  * [class_getInstanceMethod](https://developer.apple.com/documentation/objectivec/1418530-class_getinstancemethod)でメソッド情報を取得します  
  * [method_exchangeImplementations](https://developer.apple.com/documentation/objectivec/1418769-method_exchangeimplementations)でメソッドの実装を入れ替えます  
  * `method_exchangeImplementations`により`UIApplication.isNetworkActivityIndicatorVisible`が実行されたら、`ft_setNetworkActivityIndicatorVisible(visible:)`が呼び出されるという状況を作ることができます  

### まとめ
以上をまとめます。  

* iPhoneX系端末では従来の`NetworkIndicator`はデフォルト、表示されない  
* 従来型の通信表現を取り入れているアプリは最近少ないかもしれない  
* 従来型の通信表現を利用する場合は`FTLinearActivityIndicator`がオススメ  

ということで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
