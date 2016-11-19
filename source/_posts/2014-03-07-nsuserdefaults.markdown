---
layout: post
title: "NSUserDefaultsで手軽にデータを端末保存しよう！"
date: 2014-03-07 22:59
comments: true
categories: ios
---

###最も簡単なデータの端末保存方法を学ぼう！
さて、今日はNSUserDefaultsを使った簡単なデータの保存方法について紹介したいと思います。と言っても、本当に多くのサイトで十分にわかりやすい説明が書かれているので、要点を絞って簡単に説明したいと思います。

<!--more-->

まずは新規にデータを保存する場合です。  
NSString型のデータを保存する場合の例を下記に記します。  
```objective-c
// NSUserDefaultsの取得
NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];

// データの保存処理
[defaults setObject:@"value" forKey:@"key"];

// 保存の即時反映
[defaults synchronize];
```

最後のsynchronizeを利用しないとデータ保存までにタイムラグが発生する可能性があります。データ保存後に、すぐさまデータを利用する機会がある場合などはsynchronizeを呼び出した方が良いでしょう。

さて、次にデータを取得する場合です。  
```objective-c
// NSUserDefaultsの取得
NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];

// データの取得
NSString *value = [defaults stringForKey:@"key"];
```
非常に簡単にできますね。  

さて、NSUserDefaultsに保存したデータはどこにいったのでしょうか？  
実はXcodeから確認することができます。確認方法は以下の手順です。  
1. 『Window > Organizer > Applications』から確認したいアプリを選択  
2. フッターに位置するDownloadを選択  
3. Downloadしてきたファイルを右クリックして、『パッケージの内容を表示』を選択  
4. AppData > Library > Preferences > Bundle Identifier名称.plistを見る  

該当ファイルを見るとXML書式でデータが書かれていることがわかります。  
![NSUserDefaultsに保存したデータの確認](/images/nsuserdefaults.png)  

因みにNSUserDefaultsは万能ではありません。  
用途としては  
**ユーザID, プロファイルデータなどの量が増えないデータ**  
の保存が考えられます。  

もし、**『大量のデータを保存したい』、『ある条件を満たすデータのみを取得したい』**などの使い方をするのであればSQLiteを使うべきでしょう。  

ということで本日はここまで。  
