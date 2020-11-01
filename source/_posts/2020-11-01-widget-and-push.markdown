---
layout: post
title: "Push通知の受信起因でWidgetを更新しよう！"
date: 2020-11-01 13:59
comments: true
categories: ios ios14 widget push
---

### はじめに
今回は、iOS14から導入された[Widget](https://developer.apple.com/jp/widgets/)を題材として扱いたいと思います。  

既に様々なところで `Widget` の導入について説明されていますが、筆者が気になったのは、
『プッシュ通知の受信起因で `Widget` を更新することができるかどうか 』  
です。  

簡単にサンプルを作成して試してみた結果を備忘録として載せておきたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### Widgetの更新方法
まず、 `Widget` の更新方法ですが、これは大きく分けて2種類あります。  

* `Widget` 自体に定義した更新タイミングで更新する  
* アプリなどから任意のタイミングで更新する  

前者の『 `Widget` 自体に定義した更新タイミングで更新する』は、  
`Widget` ソース内の `getTimeline` メソッドが該当します。  

例えば、1時間ごとに更新したいと思った場合には、  

```objective-c
import WidgetKit
import SwiftUI
import Intents

struct Provider: IntentTimelineProvider {
  ...
  func getTimeline(for configuration: ConfigurationIntent, in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
      var entries: [SimpleEntry] = []

      // Generate a timeline consisting of five entries an hour apart, starting from the current date.
      let currentDate = Date()
      for hourOffset in 0 ..< 5 {
          let entryDate = Calendar.current.date(byAdding: .hour, value: hourOffset, to: currentDate)!
          let entry = SimpleEntry(date: entryDate, randomNumber: Int.random(in: 1 ... 10), configuration: configuration)
          entries.append(entry)
      }

      let timeline = Timeline(entries: entries, policy: .atEnd)
      completion(timeline)
  }
  ...
}

struct SimpleEntry: TimelineEntry {
  let date: Date
  let randomNumber: Int
  let configuration: ConfigurationIntent
}
```

のように定義することで実現できます。  

続いて後者の『アプリなどから任意のタイミングで更新する』は、  
任意のタイミングで、  

```objective-c
WidgetCenter.shared.reloadAllTimelines()
```

を呼び出してやれば良いだけです。  

では、プッシュ通知の受信起因で `Widget` を更新するためには、どうすれば良いのでしょうか？  

### プッシュ受信契機でWidgetを更新する
答えは、 `NotificationService Extension` を利用する方法です。  

`NotificationService Extension` はプッシュ通知受信時に条件次第で通知内容を変更したい場合などに利用されます。  

`NotificationService Extension` は下記のようにプッシュ通知の受信を検知してロジックを組み込むことができます。  

```objective-c
import UserNotifications
import WidgetKit

class NotificationService: UNNotificationServiceExtension {

    var contentHandler: ((UNNotificationContent) -> Void)?
    var bestAttemptContent: UNMutableNotificationContent?

    // プッシュ通知を受信したら、ココを通ります。
    override func didReceive(_ request: UNNotificationRequest, withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
        self.contentHandler = contentHandler
        bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)

        if let bestAttemptContent = bestAttemptContent {
            // Modify the notification content here...
            bestAttemptContent.title = "\(bestAttemptContent.title) [modified]"

            contentHandler(bestAttemptContent)
        }
    }

    override func serviceExtensionTimeWillExpire() {
        if let contentHandler = contentHandler, let bestAttemptContent =  bestAttemptContent {
            contentHandler(bestAttemptContent)
        }
    }
}
```

この `didReceive(_:withContentHandler:)` 内で `WidgetCenter.shared.reloadAllTimelines()` を実行してやれば `Widget` が更新できます。  

### (補足)プッシュ通知を簡単に検証する方法について
今回の検証では、[node-apn](https://github.com/node-apn/node-apn)を利用しました。  
`node-apn` を使えば、プッシュ通知に必要な `AuthKey` を用意し、多少の実装をするだけ簡単にプッシュ通知を検証できるのでオススメです。  

実装は、[プッシュ通知をnode-apnで送ってみよう！](https://grandbig.github.io/blog/2017/09/18/node-apn/)で詳しく説明していますので、ご参照ください。  
※ `NotificationService Extension` を利用するので、ペイロードに `"mutable-content": 1` が必要になることだけ注意してください。  

```javascript
var note = new apn.Notification();
note.badge = 3;
note.sound = "ping.aiff";
note.alert = "プッシュ通知が届きましたよ！";
note.topic = "com.xxxx.WidgetSample";
note.mutableContent = 1 // ココで "mutable-content": 1 を指定しています。
```

### まとめ
さて如何でしたでしょうか？  
`Widget` はiOS14から導入され、ホーム画面に多大な影響を与える期待の新機能ですので、  
ユーザに更なる利便性を与えるためにも積極的に導入していきたいですね。  

といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
