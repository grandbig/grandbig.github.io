---
layout: post
title: "iOS13から利用できるBackgroundTasksを使ってみよう！"
date: 2019-09-22 17:09
comments: true
categories: ios ios13 swift
---

### はじめに
今回はiOS13で新たに追加された `BackgroundTasks Framework` について見ていきたいと思います。  
基本的には、 [WWDC2019動画の『Advances in App Background Execution』](https://developer.apple.com/videos/play/wwdc2019/707/)を見ながら実践してみました。     

ですが、微妙に躓くところもあったのでメモとして残しておきたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### BackgroundTasksとは
まず、 `BackgroundTasks` の説明です。  
`BackgroundTasks` はiOS13から利用できる新しいFrameworkになります。  
`BackgroundTasks` には大きく分けて下記2つのAPIが存在します。  

1. Background Processing Tasks  
2. Background App Refresh Tasks  

#### Background Processing Tasks
`Background Processing Tasks` は以下シーンでの利用が想定されています。  

* (急を要しない)後々の実行で良いメンテナンス処理  
* Core MLを利用した機械学習のトレーニング処理  

そのため、 **数分間** の処理実行が許されています。  
また、 `requiresExternalPower` というフラグを `true` にすることで、CPU消耗によるプロセスキルをさせないよう制御することができます。  
(iOSの世界でこれって結構スゴイ気がしますね。)  

#### Background App Refresh Tasks
`Background App Refresh Tasks` は以下シーンでの利用が想定されています。  

* 30秒以内で完了できる処理  
* 1日を通してアプリを最新に保つために必要な処理  

これまで上記のような対応をする際には `Background Fetch` を利用していたことと思いますが、  
今回の新APIの発表により、旧APIはdeprecatedになったそうです。  

```objective-c
// deprecated対象
UIApplication.setMinimumBackgroundFetchInterval(_:)
UIApplicationDelegate.application(_:performFetchWithCompletionHandler:)
```

### BackgroundTasksの使い方
続いて具体的な使い方を見ていきます。  

#### ソースコードを書き始めるまでの下準備

**① Xccode上でCapabilityを追加します**  
バックグラウンド処理を利用する場合はこれまで通り `Capability` の `Background Modes` が必要になります。  
Xcode11で追加する方法が少々変わっているので気をつけましょう。  

![Background Modesを追加します](/images/backgroundtasks_1.png)  

**② Background Modesにチェックを入れます**  
今回は、 `Background Processing Tasks` と `Background App Refresh Tasks` なので下図の通りです。    

![必要なBackground Modesにチェックを入れます](/images/backgroundtasks_2.png)  

**③ Info.plistにIdentifierを登録します**  
`Info.plist` に `Permitted background task scheduler identifiers` を追加します。  
また、 `Background Processing Tasks` と `Background App Refresh Tasks` 用にそれぞれ `Identifier` を定義します。  

因みに、この `Identifier` は例によってユニークであることが求められるので、  
`com.xxxx.XXXXSample.process` , `com.xxxx.XXXXSample.refresh` といったDNSの逆書きが推奨されています。  

![Info.plistに必要なIdentifierを登録します](/images/backgroundtasks_3.png)  

#### ソースコードの実装

ここまででソースコード以外の準備は完了です。  
続いて、ソースコードを書いていきましょう。  

`Background Processing Tasks` と `Background App Refresh Tasks` それぞれ記載します。  

##### Background Processing Tasks

**① BackgroundTasksをimportします**  

```objective-c
// AppDelegate.swift
import UIKit
import BackgroundTasks
```

**② didFinishLaunchWithOptions内にバックグラウンドタスクを登録します**  

```objective-c
// AppDelegate.swift
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    ...
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.

        // 第一引数: Info.plistで定義したIdentifierを指定
        // 第二引数: タスクを実行するキューを指定。nilの場合は、デフォルトのバックグラウンドキューが利用されます。
        // 第三引数: 実行する処理
        BGTaskScheduler.shared.register(forTaskWithIdentifier: "com.MeasurementSample.refresh", using: nil) { task in
            // バックグラウンド処理したい内容 ※後述します
            self.handleAppProcessing(task: task as! BGProcessingTask)
        }
        return true
  }
}
```

**③ バックグラウンドタスクにスケジューリングします**  

```objective-c
private func scheduleAppProcessing() {
    // Info.plistで定義したIdentifierを指定
    let request = BGProcessingTaskRequest(identifier: "com.MeasurementSample.refresh")
    // 通信が発生するか否かを指定
    request.requiresNetworkConnectivity = false
    // CPU監視の必要可否を設定
    request.requiresExternalPower = true

    do {
        // スケジューラーに実行リクエストを登録
        try BGTaskScheduler.shared.submit(request)
    } catch {
        print("Could not schedule app processing: \(error)")
    }
}

func applicationDidEnterBackground(_ application: UIApplication) {
    // バックグラウンド起動に移ったときにルケジューリング登録
    scheduleAppProcessing()
}
```

**④ 実際に実行する処理を定義します**  

```objective-c
// AppDelegate.swift
// サンプル用のOperation
class PrintOperation: Operation {
    let id: Int

    init(id: Int) {
        self.id = id
    }

    override func main() {
        print("this operation id is \(self.id)")
    }
}

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
  ...
  private func handleAppProcessing(task: BGProcessingTask) {
      // 1日の間、何度も実行したい場合は、1回実行するごとに新たにスケジューリングに登録します
      scheduleAppRefresh()

      let queue = OperationQueue()
      queue.maxConcurrentOperationCount = 1

      // 時間内に実行完了しなかった場合は、処理を解放します
      // バックグラウンドで実行する処理は、次回に回しても問題ない処理のはずなので、これでOK
      task.expirationHandler = {
          queue.cancelAllOperations()
      }

      // サンプルの処理をキューに詰めます
      let array = [1, 2, 3, 4, 5]
      array.enumerated().forEach { arg in
          let (offset, value) = arg
          let operation = PrintOperation(id: value)
          if offset == array.count - 1 {
              operation.completionBlock = {
                  // 最後の処理が完了したら、必ず完了したことを伝える必要があります
                  task.setTaskCompleted(success: operation.isFinished)
              }
          }
          queue.addOperation(operation)
      }
  }
}
```

##### Background App Refresh Tasks

**① BackgroundTasksをimportします**  

```objective-c
// AppDelegate.swift
import UIKit
import BackgroundTasks
```

**② didFinishLaunchWithOptions内にバックグラウンドタスクを登録します**  

```objective-c
// AppDelegate.swift
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    ...
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.

        // 第一引数: Info.plistで定義したIdentifierを指定
        // 第二引数: タスクを実行するキューを指定。nilの場合は、デフォルトのバックグラウンドキューが利用されます。
        // 第三引数: 実行する処理
        BGTaskScheduler.shared.register(forTaskWithIdentifier: "com.MeasurementSample.refresh", using: nil) { task in
            // バックグラウンド処理したい内容 ※後述します
            self.handleAppRefresh(task: task as! BGAppRefreshTask)
        }
        return true
  }
}
```

**③ バックグラウンドタスクにスケジューリングします**  

```objective-c
private func scheduleAppRefresh() {
    // Info.plistで定義したIdentifierを指定
    let request = BGAppRefreshTaskRequest(identifier: "com.MeasurementSample.refresh")
    // 最低で、どの程度の期間を置いてから実行するか指定
    request.earliestBeginDate = Date(timeIntervalSinceNow: 15 * 60)

    do {
        // スケジューラーに実行リクエストを登録
        try BGTaskScheduler.shared.submit(request)
    } catch {
        print("Could not schedule app refresh: \(error)")
    }
}

func applicationDidEnterBackground(_ application: UIApplication) {
    // バックグラウンド起動に移ったときにルケジューリング登録
    scheduleAppRefresh()
}
```

**④ 実際に実行する処理を定義します**  

```objective-c
// AppDelegate.swift
// サンプル用のOperation
class PrintOperation: Operation {
    let id: Int

    init(id: Int) {
        self.id = id
    }

    override func main() {
        print("this operation id is \(self.id)")
    }
}

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
  ...
  private func handleAppRefresh(task: BGAppRefreshTask) {
      // 1日の間、何度も実行したい場合は、1回実行するごとに新たにスケジューリングに登録します
      scheduleAppRefresh()

      let queue = OperationQueue()
      queue.maxConcurrentOperationCount = 1

      // 時間内に実行完了しなかった場合は、処理を解放します
      // バックグラウンドで実行する処理は、次回に回しても問題ない処理のはずなので、これでOK
      task.expirationHandler = {
          queue.cancelAllOperations()
      }

      // サンプルの処理をキューに詰めます
      let array = [1, 2, 3, 4, 5]
      array.enumerated().forEach { arg in
          let (offset, value) = arg
          let operation = PrintOperation(id: value)
          if offset == array.count - 1 {
              operation.completionBlock = {
                  // 最後の処理が完了したら、必ず完了したことを伝える必要があります
                  task.setTaskCompleted(success: operation.isFinished)
              }
          }
          queue.addOperation(operation)
      }
  }
}
```

### BackgroundTasksのデバッグ方法
上記で実装が完了しました。  
実際に挙動を試すためには、特別な手順が必要になります。  

① アプリを起動します  
② アプリをバックグラウンドに移します(登録トリガーのためです)  
③ 再度アプリを起動します  
④ Xcodeで `Pause Program Execution` をタップします  

![Pause Program Executionをタップします](/images/backgroundtasks_4.png)  

⑤ LLDBに以下コマンドを入力して実行します  

```objective-c
// Background Processing Tasksの場合
e -l objc -- (void)[[BGTaskScheduler sharedScheduler] _simulateLaunchForTaskWithIdentifier:@"com.MeasurementSample.process"]

// Background App Refresh Tasksの場合
e -l objc -- (void)[[BGTaskScheduler sharedScheduler] _simulateLaunchForTaskWithIdentifier:@"com.MeasurementSample.refresh"]
```

⑥ 再度、Xcodeで `Pause Program Execution` をタップします  

以上で実際の実行処理が見れるようになるはずです。  

因みに、実行処理の期限切れを試したい場合は、以下のように⑤のLLDBで以下を入力して実行します  

```objective-c
// Background Processing Tasksの場合
e -l objc -- (void)[[BGTaskScheduler sharedScheduler] _simulateExpirationForTaskWithIdentifier:@"com.MeasurementSample.process"]

// Background App Refresh Tasksの場合
e -l objc -- (void)[[BGTaskScheduler sharedScheduler] _simulateExpirationForTaskWithIdentifier:@"com.MeasurementSample.refresh"]
```

### ハマったところ
筆者が試しに実装してハマったところを参考までに載せておこうと思います。  
筆者の場合、なぜかシミュレータで実行しようとすると、以下のエラーが発生してしまいました。  

```objective-c
BGTaskSchedulerErrorDomain Code=1 "(null)"
```

これは、[サンプルコード](https://developer.apple.com/documentation/backgroundtasks/refreshing_and_maintaining_your_app_using_background_tasks)で試しても同様でした。  
ただ、実機で試したところ問題なく通ったんですよね...  

### まとめ
さて如何でしたでしょうか。  
iOS13で追加された新APIは旧バージョンサポートのため、
すぐには利用されないかもしれませんが、iOS13の普及に伴い、利用シーンは確実に増えていくことでしょう。  
そのため、サンプル程度の実装でも、使い方を学んでおくことは今後の役に立つと思っています。  

と言ったところで本日はここまで。  

* 参考URL:  
  * [WWDC2019 - Advances in App Background Execution](https://developer.apple.com/videos/play/wwdc2019/707/)  
  * [Apple Documentation - BackgroundTasks](https://developer.apple.com/documentation/backgroundtasks)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
