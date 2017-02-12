---
layout: post
title: "Notificationとメインスレッド/サブスレッド"
date: 2017-02-12 17:15
comments: true
categories: ios swift
---

###はじめに
今日は久々にiOS関連の話を書きたいと思います。  
正直なところSwiftを触って思ったのですが、今やXcodeではSwift3が標準装備されているため書き方が変わっており、業務で触らないまでも遅れを取らないように勉強しなきゃな...なんて感じてしまいました。  

本日のお題は  
『`Notification`はメインスレッド/サブスレッドのどちらで実行されるのか』  
という話です。  

筆者の理解では、普通に使う限りメインスレッドだと思っていたのですが、「使い方に寄るのでは？」なんて話題が上がったこともあって、調べてみたくなりました。  
その調べ方と調査結果を書いていきます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

###Notificationとは
まずは、`Notification`とは何かという話から始めましょう。  
その名の通り、アプリ内で通知を行う仕組みを実装するために利用します。  
少し前までは`NSNotification`という名前でしたので、こちらの名前の方がしっくりくるiOSエンジニアの方々は多いのではないでしょうか。  

使い方は非常に簡単です。  

１．監視対象のメソッドを登録する  

![監視対象のメソッドを登録](/images/notification_1.png)  

```objective-c
// 登録時にはaddObserverメソッドを利用する
NotificationCenter.default.addObserver(self, selector: #selector(self.doSomething), name: NSNotification.Name("SomeNotification"), object: nil)
```

２．監視対象メソッドの呼び出しを要求する  

![監視対象メソッドの呼び出し要求](/images/notification_2.png)  

```objective-c
// 呼び出し要求時にはpostメソッドを利用する
NotificationCenter.default.post(name: NSNotification.Name("SomeNotification"), object: nil)
```

３．監視対象メソッドを実行する

![監視対象メソッドを実行](/images/notification_3.png)  

###Notificationとメインスレッド/サブスレッドの調査
さて、本題に入ります。  
`Notification`はメインスレッド/サブスレッドどちらで実行されるのでしょうか。  

####普通に呼び出す場合
まずは特異なことをせず、普通に実行してみます。  
要件としては下記になります。

* アプリ起動時に`doSomething`メソッドを`NotificationCenter`に登録する  
* ボタンタップ時に`doSomething`メソッドの呼び出しを要求する  
* `doSomething`メソッド内でメインスレッド/サブスレッドの判定処理を実行する  

ソースコードは下記となります。  

```objective-c
// ViewController.swift
import UIKit

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        NotificationCenter.default.addObserver(self, selector: #selector(doSomething), name: NSNotification.Name("SomeNotification"), object: nil)
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }

    func doSomething() {
        print("このメソッドは\(Thread.isMainThread)で動いています。(false: サブスレッド, true: メインスレッド)")
    }
    @IBAction func checkThread(_ sender: Any) {
        NotificationCenter.default.post(name: NSNotification.Name("SomeNotification"), object: nil)
    }
}
```

結果は「呼び出し先の処理実行は **メインスレッド** 」となりました。  

####GCDを利用して、サブスレッド内でメソッドを呼び出す
次はサブスレッドで監視対象メソッドの呼び出し要求を実施します。  
ソースコードは下記の通りです。  

```objective-c
import UIKit

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        NotificationCenter.default.addObserver(self, selector: #selector(doSomething), name: NSNotification.Name("SomeNotification"), object: nil)
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }

    func doSomething() {
        print("このメソッドは\(Thread.isMainThread)で動いています。(0: サブスレッド, 1: メインスレッド)")
    }
    @IBAction func checkThread(_ sender: Any) {
        DispatchQueue.global().async {
            NotificationCenter.default.post(name: NSNotification.Name("SomeNotification"), object: nil)
        }
    }
}
```

結果は「呼び出し先の処理実行は **サブスレッド** 」となりました。  

####GCDを利用して、メインスレッド内でメソッドを呼び出す
今度はGCDを利用したメインスレッド内で監視対象メソッドの呼び出し要求を実施してみます。  
ソースコードは下記の通りです。  

```objective-c
import UIKit

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()NotificationCenter.default.addObserver(self, selector: #selector(doSomething), name: NSNotification.Name("SomeNotification"), object: nil)
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }

    func doSomething() {
        print("このメソッドは\(Thread.isMainThread)で動いています。(0: サブスレッド, 1: メインスレッド)")
    }
    @IBAction func checkThread(_ sender: Any) {
        DispatchQueue.global().async {
            DispatchQueue.main.async {
                NotificationCenter.default.post(name: NSNotification.Name("SomeNotification"), object: nil)
            }
        }
    }
}
```

結果は「呼び出し先の処理実行は **メインスレッド** 」となりました。  

###まとめ
調査結果をまとめると次の通りです。  

「要求 **元** と **同じスレッド** で監視対象メソッドは実行されます。」  
※登録時(`addObserver`)のスレッドは無関係です。  

さて如何でしたでしょうか？  
普段考えずに利用している様々な処理が実際にはどのスレッドで実行されるのか、どういった書き方をするとどのスレッドで実行されうるのか調べてみることも面白いですね。  
ふとしたときにまた調べてみようと思います。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
