---
layout: post
title: "Firebase NotificationをiOSで使ってみよう！"
date: 2017-09-18 00:45
comments: true
categories: ios swift firebase google
---

### はじめに
今回はFirebase Notificationについて見ていきたいと思います。  
一昔前であれば、プロダクトごとにNotificationの仕組みを作り込んだり、共通基盤としてNotificationプロジェクトを推進していたりといった会社が多かった気がします。  
また、未だにリッチなNotificationのプロダクトを生業として利益を上げている会社もあるので、それだけNotificationの仕組みは自作ではなくあるものを使いたいという需要が大きいのでしょう。  
しかしながら、Firebaseの登場により、Notificationプロダクト市場もより加熱化しているのではないでしょうか。  

筆者的にはGoogleが出す、それも無料のプロダクトであるわけなので、使わない手はないと思うわけです。  
なんて偉そうなことを言いつつ、FireabseによるNotificationの仕組みを利用したことがなかったので、今回試しに使ってみることにしました。  


<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### Firebaseの導入
こちらの内容は[以前書いた記事](https://grandbig.github.io/blog/2017/05/14/firebase-ios/)に任せたいと思います。  

### APNs証明書の作成
こちらに関しては、様々なサイトにて説明がされているので、そちらを見るなどした方が早いので省きます。  
例えば [プッシュ通知に必要な証明書の作り方2017](http://qiita.com/natsumo/items/d5cc1d0be427ca3af1cb#6-apns%E7%94%A8%E8%A8%BC%E6%98%8E%E6%9B%B8cer%E3%81%AE%E4%BD%9C%E6%88%90)など参考にすると良いでしょう。  

### Firebase Notificationの利用
さて、本題です。  
Firebase Notificationを利用するための手順について説明します。  

① 作成したAPNs証明書をダウンロードする  
先程作成したAPNs証明書はキーチェーンに登録していると思います。  
(Apple Developer Programで作成したAPNs証明書は必ずDLしてキーチェーンに登録しましょう。)  
キーチェーンを開いて、該当の証明書をダウンロードします。  

該当のAPNs証明書は `Apple Development IOS Push Services: xxxxx` のようになっているものです。  
![キーチェーンを開く](/images/firebase_notificcation_ios_1.png)  

右クリックして書き出すを選択します。  
![証明書を書き出す](/images/firebase_notificcation_ios_2.png)  

ファイル名とパスワードをつけて保存します。  
![ファイル名とパスワードの付け方](/images/firebase_notificcation_ios_3.png)  

② ダウンロードしたAPNs証明書をFirebaseに登録する  

FirebaseのConsoleを開き、設定画面に遷移します。  
![Firebase Consoleの設定画面](/images/firebase_notificcation_ios_4.png)  

クラウドメッセージングタブを開き、APNs証明書をアップロードします。  
![Firebase Console クラウドメッセージングタブ](/images/firebase_notificcation_ios_5.png)  

③ Firebaseの必要ライブラリを導入する  
Firebaseの導入で`Core`ライブラリはインストールできているかもしれませんが、Notificationでは`Messaging`が必要になります。  
よって、`Podfile`を下記のように修正して、`pod update`をする必要があります。  

```objective-c
# Podfile
use_frameworks!

target "NotificationSample" do
  # Normal libraries
  pod 'Firebase/Core'
  pod 'Firebase/Messaging'

  abstract_target 'Tests' do
    inherit! :search_paths
    target "NotificationSampleTests"
    target "NotificationSampleUITests"
  end
end
```

④ ソースコードにFirebase Notificationが利用できるように実装する  
続いて、ソースコードの設定です。

```objective-c
// AppDelegate.swift
import UIKit
import Firebase
import UserNotifications

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate, UNUserNotificationCenterDelegate, MessagingDelegate {
  var window: UIWindow?
  func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    // リモート通知 (iOS10に対応)
    let authOptions: UNAuthorizationOptions = [.alert, .badge, .sound]
    UNUserNotificationCenter.current().requestAuthorization(
      options: authOptions,
      completionHandler: {_, _ in })

    // UNUserNotificationCenterDelegateの設定
    UNUserNotificationCenter.current().delegate = self
    // FCMのMessagingDelegateの設定
    Messaging.messaging().delegate = self

    // リモートプッシュの設定
    application.registerForRemoteNotifications()
    // Firebase初期設定
    FirebaseApp.configure()

    // アプリ起動時にFCMのトークンを取得し、表示する
    let token = Messaging.messaging().fcmToken
    print("FCM token: \(token ?? "")")

    return true
  }

  // 省略

  func userNotificationCenter(_ center: UNUserNotificationCenter,
                                willPresent notification: UNNotification,
                                withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
    print("フロントでプッシュ通知受け取ったよ")
  }

  func messaging(_ messaging: Messaging, didRefreshRegistrationToken fcmToken: String) {
    print("Firebase registration token: \(fcmToken)")
  }
}
```

たったのこれだけでプッシュ通知の受信ができるようになります。  

### Firebase Notificationを利用する
設定が完了したため、Firebaseからプッシュ通知を送ってみましょう。  

方法は簡単です。  

① 左メニューから `Notification` を選択する  
② 「新しいメッセージ」を選択する  
③ 必要項目を入力して「メッセージを送る」を選択する  

![Firebase Consoleからプッシュ通知を送る](/images/firebase_notificcation_ios_6.png)  

これにより、下図のようにプッシュが届くことを確認できます。  
![端末にプッシュ通知が届く](/images/firebase_notificcation_ios_7.png)  

### FirebaseにAPNs Keyを登録する方法
さて、上記まででは、APNs証明書を作成して、Firebaseに登録する方法を説明しましたが、実はAPNs証明書よりも `APNs Key` というものの利用を推奨されています。  
なので、こちらの方法も説明しておきます。  

作成方法はいたって簡単です。  

① Apple Developer Programの左メニューから`Keys > All`を選択する  
② 「＋」ボタンで新規作成する  
③ Keyの名称を設定する  
④ Service種別として`APNs`を選択する  
⑤ Continueボタンを選択して作成する  

![APNs Keyの作成方法](/images/firebase_notificcation_ios_8.png)  

上記で作成した`APNs Key`をFirebase Console上で設定します。  

Firebase Console > 設定 > クラウドメッセージング > iOSアプリの設定から`APNs認証キー`に設定します。  

![APNs認証キーを設定](/images/firebase_notificcation_ios_9.png)  

因みに、設定時に利用するIDはそれぞれ下記から取得します。  

![キーID](/images/firebase_notificcation_ios_10.png)  
![App ID Prefix](/images/firebase_notificcation_ios_11.png)  


### まとめ
さて如何でしたでしょうか？  
まだまだFirebase Console上からプッシュできるNotificationにも制限があるようですが、複雑なことをしない例えば「一斉お知らせ機能」のような形で利用するのであれば非常に良いのではないでしょうか？  
今後もバージョンアップして使いやすくなることを期待しつつ、本日はここまで。  


<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
