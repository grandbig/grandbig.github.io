---
layout: post
title: "Crashlyticsが作成するエラーファイルを見てみよう！"
date: 2018-04-29 17:44
comments: true
categories: ios swift crashlytics
---

### はじめに
今回は[Fabric](https://fabric.io/home)のメイン機能である `Crashlytics` について紹介したいと思います。  
筆者が初めて `Crashlytics` を利用したのは、もう数年前になるでしょうか。  
それこそ、当時は `Fabric` という名称ではなく、本当にただの `Crashlytics` というサービス名だったんですよね。  

あの頃は、クラッシュレポートを溜める独自サービスが大量生産されていたのではないでしょうか？  
そんな市場に風穴を開けたのが、 `Crashlytics` でした。  
なんてったって、無料なんですから、使わない手はなかったわけです。  
更に今や `Firebase` だったり、`fastlane` だったりと連携しちゃうわけですからね。  

前置きはこのくらいにして、  
今回は `Crashlytics` の `crash()` や `recordError()` について少々調べたことを書き留めておきたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### Crashlyticsの導入
まずは、`Crashlytics`の導入ですね。  
[公式ページ](https://fabric.io/kits/ios/crashlytics/install)に従って進めれば簡単に準備ができることでしょう。  
注意点としては、`Fabric` のダッシュボードでクラッシュレポート数や内容を見るために、`dSYM`ファイルをアップロードする必要があるということくらいです。  

その手順を見ていきましょう。  

① Xcode > PROJECT or TARGET > Build Settings > Build Options > Debug Information Formatの設定を変更する  

開発版では`Debug`モードで`Run`していることもあると思うので、  
その場合は下図のように `DWARF with dSYM File` に変更しましょう。  

![Debug Information FormatをDWARF with dSYM Fileに変更する](/images/crashlytics_5.png)  

② Xcode > Product > Archive を実行する  

③ dSYMをDownloadする  

Organizer Windowが自動で表示されるため、dSYMファイルをダウンロードしましょう。  
Archive Informationの『Download dSYMs...』を選択するか、  
もしくは下図のように該当アプリを右クリックして『Show in Finder』を選択して直接Downloadしましょう。  

![Show in Finderを選択する](/images/crashlytics_6.png)  

④ Show in Findeで開いたフォルダのdSYMsフォルダをzip化しましょう。  

![dSYMsフォルダをzip化する](/images/crashlytics_7.png)  

⑤ dSYMs.zipをFabric管理画面からUploadしましょう。  

Settings > APPs > 該当アプリ > Missing DSYMs からDrag&Dropでアップロードできます。  

![dSYMs.zipのアップロード](/images/crashlytics_8.png)  

これでクラッシュ後に、アプリを再起動して送信されたレポートを`Fabric`管理画面から確認することができます。  

### crash()で生成されるファイル
`Crashlytics`で最も利用されているであろうクラッシュ時のレポート送信ですが、  
一体、何を送っているのでしょうか？  

今回は調査のために、わざとクラッシュさせて、アプリ内の『どこに』『何が』保存されているのか見てみました。  

#### crash()メソッドの呼び出し
以下のようにボタンタップしたら、クラッシュするようにしてみましょう。  

```objective-c
// ViewController.swift
@IBAction func tappedCrashButton(_ sender: Any) {
    Crashlytics.sharedInstance().crash()
}
```

#### クラッシュで生成されるファイルの確認方法
クラッシュ後の生成ファイルの確認方法は以下です。  

① アプリを起動してボタンタップでクラッシュさせます。  
このとき重要なのが、『デバッグ状態にはせずにクラッシュさせる』ということです。  
(どうもXcodeに接続したデバッグ状態だと、うまく記録されないようです。)  

② 端末をMacに繋ぐ  

③ Xcode > Window > Devices and Simulators を選択する  
これにより以下のようなWindowが表示されます。  
![Devices and Simulators Window](/images/crashlytics_1.png)

④ Downloadしたいアプリ内ファイルを選択して「Download Container...」を選択する  
![Download Container...を選択する](/images/crashlytics_2.png)

⑤ 自動でDownload先フォルダが表示されるので、該当ファイルを右クリックして、「パッケージの内容を表示」を選択する  
![Downloadファイルの内容を表示する](/images/crashlytics_3.png)  

⑥ クラッシュログを確認する  

```
xxxxxx.xcappdata/AppData/Library/Caches/com.crashlytics.data/<BundleID>/v3/active/xxxxxxxxxxx/match_exception.clsrecord
```

がそれに該当します。  

![クラッシュログの格納場所](/images/crashlytics_4.png)  

#### クラッシュログの中身
中身を見てみると、  

```objective-c
{
  "mach_exception":{
    "exception":6,
    "codes":[1,4332784292],
    "name":"EXC_BREAKPOINT",
    "code_name":null,
    "original_ports":1,
    "time":1525070032
  }
}
{
  "threads":[
  {
    "registers":{
      "x0":7851893440,"x1":4337440564,"x2":4431287584,"x3":7651498240,"x4":7651498240,"x5":7651498240,"x6":7516198016,"x7":5305,"x8":7851893440,"x9":0,"x10":80378702350618752,"x11":18714625,"x12":18714368,"x13":0,"x14":18714625,"x15":18714816,"x16":6457744324,"x17":4332784292,"x18":0,"x19":7651498240,"x20":4431355264,"x21":4337440648,"x22":4431355264,"x23":4431355264,"x24":7516197984,"x25":0,"x26":6639092750,"x27":1,"x28":7854100944,"fp":6134339024,"sp":6134338992,"lr":4332705516,"pc":4332784292,"cpsr":1610612736
    },
    "stacktrace":[4332784292,4332705516,4332705636,6626567880,6627752100,6626592636,6627860956,6627105352,6627059960,6627054136,6635318284,6635327928,6635328792,6635299416,6461387780,6461385772,6461376412,6460460456,6493892640,6628104076,4332687252,6454763456],
  "crashed":true
    },
  {
    "registers":{
      "x0":256,"x1":6136048512,"x2":1,"x3":0,"x4":9991,"x5":0,"x6":0,"x7":0,"x8":7852716416,"x9":1,"x10":7852716472,"x11":236223201281,"x12":0,"x13":0,"x14":1,"x15":7,"x16":368,"x17":48,"x18":0,"x19":6136049664,"x20":7852716416,"x21":33,"x22":1,"x23":7303651328,"x24":7303651328,"x25":0,"x26":7,"x27":2164260864,"x28":1,"fp":6136048352,"sp":6136048192,"lr":6457704116,"pc":6456008068,"cpsr":1073741824
    },
    "stacktrace":[6456008068,6457704116,6457703176]
  }
  ...
  ]
}
{"runtime":{"objc_selector":"","crash_info_entries":[]}}
{"dispatch_queue_names":["com.apple.main-thread","","","","","","","",""]}
{"thread_names":["","","com.apple.uikit.eventfetch-thread","","com.twitter.crashlytics.ios.MachExceptionServer","","com.google.Maps.LabelingBehavior","com.apple.NSURLConnectionLoader",""]}
{"process_stats":{"active":1085800448,"inactive":460079104,"wired":669319168,"freeMem":116162560,"free_mem":116162560,"virtual":2060877824,"resident":1085800448,"user_time":337647,"sys_time":0}}
{"storage":{"free":206087524352,"total":255937040384}}
```

のようになっています。  
これがいい具合に `Fabric` 管理画面に表示されるわけですね。  

### recordError()で生成されるファイル
クラッシュではないけども、原因調査のためにレポート送信したい時があると思います。  
そんなときは `recordError()` メソッドを利用します。  

#### recordError()メソッドの呼び出し
以下のようにボタンをタップしたら、レポートを送信するようにしてみましょう。  

```objective-c
// ViewController.swift
@IBAction func tappedCrashButton(_ sender: Any) {
    let error = NSError(domain: "takahiro test", code: 1001, userInfo: nil)
    Crashlytics.sharedInstance().recordError(error)
}
```

#### recordError()で生成されるファイルの確認方法
`recordError()` 実行後の生成ファイル確認方法は、基本的にはクラッシュ時と同じです。  
ただし、クラッシュ時に生成されるファイルとファイル名が異なります。  
`recordError()` では `error_a.clsrecord` といったファイルが生成されます。  

#### recordError()で記録したファイルの中身

記録内容は以下の通りです。  
何を格納するかで中身はもう少し変わりそうですね。  

```objective-c
{"error":{"domain":"74616b616869726f2074657374","code":1001,"time":1524979041,"stacktrace":[4299168744,4299051336,4298970328,4298970672,6639789768,6640973988,6639814524,6641082844,6640327240,6640281848,6640276024,6648540172,6648549816,6648550680,6648521304,6474609668,6474607660,6474598300,6473682344,6507114528,6641325964,4298951872,6467985344]}}
{"error":{"domain":"74616b616869726f2074657374","code":1001,"time":1524979043,"stacktrace":[4299168744,4299051336,4298970328,4298970672,6639789768,6640973988,6639814524,6641082844,6640327240,6640281848,6640276024,6648540172,6648549816,6648521304,6474609668,6474607660,6474598300,6473682344,6507114528,6641325964,4298951872,6467985344]}}
{"error":{"domain":"74616b616869726f2074657374","code":1001,"time":1524979046,"stacktrace":[4299168744,4299051336,4298970328,4298970672,6639789768,6640973988,6639814524,6641082844,6640327240,6640281848,6640276024,6648540172,6648549816,6648521304,6474609668,6474607660,6474598300,6473682344,6507114528,6641325964,4298951872,6467985344]}}
{"error":{"domain":"74616b616869726f2074657374","code":1001,"time":1524979048,"stacktrace":[4299168744,4299051336,4298970328,4298970672,6639789768,6640973988,6639814524,6641082844,6640327240,6640281848,6640276024,6648540172,6648549816,6648521304,6474609668,6474607660,6474598300,6473682344,6507114528,6641325964,4298951872,6467985344]}}
```

#### recordError()の注意点
今回、`recordError()`を利用していて気づいた点が以下の通りです。  

* 短時間に複数回タップしてもアプリ内のファイルにエラーログとして記録されない  
* アプリ内に記録されたエラーログが全て`Fabric`管理画面に記録されるわけではない  

どういったフィルタがかかっているのかまではわからないのですが、なんでもかんでも記録されるわけではないようです...  

### まとめ
さて如何でしたでしょうか？  
今回たまたま中身を探ってみる機会があったので見てみましたが、きっと普段は気にしなくて良い内容なのだと思います。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
