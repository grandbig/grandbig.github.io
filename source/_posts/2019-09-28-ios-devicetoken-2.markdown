---
layout: post
title: "iOS13におけるプッシュ通知に必要なデバイストークンの取得方法"
date: 2019-09-28 16:12
comments: true
categories: ios devicetoken notification
---

### はじめに
今回はiOS13のプッシュ通知用デバイストークンについて見ていきたいと思います。
`Swift` に関して言えば特に変更があったわけではないのですが、  
`Objective-C` において少々仕様の変更があったようです。  

具体的には後述しますが、 `description` を利用してデバイストークンを取得する方式は `iOS13` から見直す必要がありそうです。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### プッシュ通知の受信処理の実装
折角なので、基本的な設定についても説明していきます。  

#### Xcode11の設定
まずは、Xcode11上で下図の状態まで設定します。  

**▼確認観点**  
* `Team ID` が利用想定のものであること  
* `Bundle Identifier` が利用想定のものであること  
* `Signing Certificate` が利用想定のものであること  
* `Provisioning Profile` の `Capabilities` に `Push Notifications` が含まれている  
* `Capability` として `Push Notifications` が追加すること  

![Signing & Capabilities](/images/ios-devicetoken-2_1.png)  

#### AppDelegateの実装
続いて、 `AppDelegate.swift` 上の実装です。  

基本的なステップは4つです。  

① プッシュ通知の利用許可のリクエストを送信します  
② 利用許可を得た場合に、プッシュ通知の利用登録を実行します  
③ プッシュ通知の利用登録が成功したことをキャッチする `Delegate` メソッドを書きます  
④ プッシュ通知の利用登録が失敗したことをキャッチする `Delegate` メソッドを書きます  

```objective-c
import UIKit
import UserNotifications

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.

        // ① プッシュ通知の利用許可のリクエスト送信
        UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge]) { granted, error in
            guard granted else { return }

            DispatchQueue.main.async {
                // ② プッシュ通知利用の登録
                UIApplication.shared.registerForRemoteNotifications()
            }
        }

        return true
    }
    ...
}

extension AppDelegate {

    // ③ プッシュ通知の利用登録が成功した場合
    func application(_ application: UIApplication,
                     didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        let token = deviceToken.map { String(format: "%.2hhx", $0) }.joined()
        print("Device token: \(token)")
    }

    // ④ プッシュ通知の利用登録が失敗した場合
    func application(_ application: UIApplication,
                     didFailToRegisterForRemoteNotificationsWithError error: Error) {
        print("Failed to register to APNs: \(error)")
    }
}
```

### 旧来のデバイストークンの取得方法
さて本題に入ります。  
`Objective-C` 時代に主流と言われていたデバイストークンの取得方法は、以下のような `description` を利用した手法でした。  

```objective-c
NSString *token = [[[[deviceToken description] stringByReplacingOccurrencesOfString:@"<"withString:@""]
                   stringByReplacingOccurrencesOfString:@">" withString:@""]
                   stringByReplacingOccurrencesOfString: @" " withString: @""];
```

これは、 `[deviceToken description]` で取得される値が、  
`<7097978d 1e438923 ... 6fdbf111>` というような先頭と末尾を `<>` で括られていたためです。  
そして、(見やすくはないですが)1文で書けるというのも当時から魅力的だったのかもしれません。  

`Swift` に言語が変わってからも、しばらくは上記手法が多くの場面で使われてきたことと思います。  
しかし、 `Swift3` になったタイミングで、状況が変わりました。  
プッシュ通知の利用登録成功時の `Delegate` メソッドの `deviceToken` が `NSData` から `Data` になったことで、  
この手法が利用できなくなったのです。  

この時点で  

```objective-c
let token = deviceToken.map { String(format: "%.2hhx", $0) }.joined()
```

という書き方がデバイストークン取得の主流に完全に取って代わったと思います。  
しかし、あくまでもこれは `Swift` におけるデバイストークンの取得であって、 `Objective-C` は旧来の書き方が続いていました。  

最近、これがどうやら `iOS13` から `Objective-C` でも変更が出たようなんです。  

* [Apple Developer Forums: iOS13 PKPushCredentials broken](https://forums.developer.apple.com/thread/117545)  
* [Apple Developer Forums: NSData description and NSString stringWithFormat have different return results when compiled with Xcode 11 versus Xcode 10](https://forums.developer.apple.com/thread/119111)  

では、 `Objective-C` ではどう書いていくべきかと言うと、 `Facebook` が1つの解を示してくれています。  

```objective-c
// ↓ dataにDelegateメソッドに渡ってきたdeviceTokenを渡します
(NSString *)hexadecimalStringFromData:(NSData *)data  
{  
    NSUInteger dataLength = data.length;  
    if (dataLength == 0) {  
        return nil;  
    }  

    const unsigned char *dataBuffer = data.bytes;  
    NSMutableString *hexString  = [NSMutableString stringWithCapacity:(dataLength * 2)];  
    for (int i = 0; i < dataLength; ++i) {  
        [hexString appendFormat:@"%02x", dataBuffer[i]];  
    }  
    return [hexString copy];  
}
```

これで `Swift` と `Objective-C` 共に安定した書き方に変わったと言えるでしょう。  

### まとめ
さて如何でしたでしょうか。  
iOS13は細かいところも含めて様々な変更が入っているため、日々最新情報をキャッチしつつ、自身でも積極的に使い倒していく必要がありますね。  

と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
