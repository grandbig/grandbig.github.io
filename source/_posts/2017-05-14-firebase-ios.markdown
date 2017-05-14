---
layout: post
title: "FirebaseをiOSで使ってみよう！"
date: 2017-05-14 16:30
comments: true
categories: ios firebase
---

### はじめに
今や商用で利用している人も珍しくないであろうFirebaseを触ってみようと思います！  
と言うのも、最近Stackoveflowを眺めている時にFirebaseに関する質問を時たま見かけるようになったからです。  
(筆者はまともにFirebaseを使ったことがないため、この機会に使ってみようと思い立ちました。)  

ということで、初歩の初歩から見ていきたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
<!-- more -->

### Firebaseの導入
まずは何と言っても利用方法ですよね？  
ほとんどGoogleが用意してくれているため、本当に簡単にできます。  

１．Firebaseの公式ページにアクセス  
まずは、[Firebaseの公式ページ](https://console.firebase.google.com/?hl=ja)に遷移して、新規プロジェクトを作成しましょう。  

![Firebaseの新規プロジェクトを作成](/images/firebase_ios_1.png)    

２．iOSアプリにFirebaseを追加  
１で新規プロジェクトが作成できたため、続いてiOSアプリにFirebaseを追加しましょう。  

追加対象のXcodeプロジェクトを作成します。  
![Xcodeプロジェクトの作成](/images/firebase_ios_2.png)  

Firebase管理ページから「iOSアプリにFirebaseを追加」を選択します。  
![iOSアプリにFirebaseを追加](/images/firebase_ios_3.png)  

plistファイルをDLしてXcodeプロジェクトに追加します。  
![plistファイルをXcodeプロジェクトに追加](/images/firebase_ios_4.png)  

続いて、`CocoaPods`でFrameworkをインストールします。  
![CocoaPodsでFrameworkを追加](/images/firebase_ios_5.png)  

`Podfile`には下記を記載しました。  

```objective-c
# Podfile
use_frameworks!

target "FirebaseSample" do
  # Normal libraries
  pod 'Firebase/Core'

  abstract_target 'Tests' do
    inherit! :search_paths
    target "FirebaseSampleTests"
    target "FirebaseSampleUITests"
  end
end
```

そして、`.xcworkspace`ファイルを開いて、下図の指示の通り`AppDelegate.swift`にコードを追加します。  

![AppDelegate.swiftにコードを追加](/images/firebase_ios_6.png)  

```objective-c
import UIKit
import Firebase // ここを追記

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.
        FIRApp.configure()　// ここを追記
        return true
    }
}
```

これでFirebase管理画面にiOSプロジェクトとの紐付けが完了したことがわかります。  

![iOSプロジェクトへのFirebase紐付け完了](/images/firebase_ios_7.png)  

### Firebaseの導入を確認
ここまででFirebaseの導入が完了したものの、正しく導入できたかどうかわからないかと思います。  
そういうときには`DebugView`を利用しましょう。  

１．XcodeプロジェクトにFirebaseのデバッグ設定を追加  
Target > Edit Scheme > Run > Arguments を見てみましょう。  

![Firebaseデバッグ設定を追加](/images/firebase_ios_8.png)  

`-FIRDebugEnabled`をONにしておくことでデバッグが可能になります。  
デバッグ機能を利用しない場合は明示的に`-FIRDebugDisabled`をONにしましょう。  
([Google公式ページ](https://support.google.com/firebase/answer/7201382?hl=ja&utm_id=ad)によると一度デバッグ機能を有効化すると`-FIRDebugDisabled`を指定しないと無効化しないそうです。)  

２．Xcodeからデバッグ状況を確認  
上記設定を追加した状態で実機でアプリを起動してみましょう。  
すると、Xcodeのコンソールログ欄に下図のようにログが出力されるようになります。  

![Xcodeにログが出力されます](/images/firebase_ios_9.png)  

３．Firebase管理サイトでデバッグ状況を確認  
Firebase管理サイトからもデバッグ状況を確認できます。  

![Firebase管理サイトでデバッグ状況を確認](/images/firebase_ios_10.png)  

### まとめ
これでFirebaseをiOSに追加することができました。非常に簡単ですね！  
サイドメニューを見るだけでも、非常に多くの機能を有しているようなので、少しずつ試してみたいと思いますが、まずは導入まで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
