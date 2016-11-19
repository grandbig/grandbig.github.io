---
layout: post
title: "iOSアプリ内課金: 自動更新購読"
date: 2016-09-25 17:45
comments: true
categories: ios in-app-purchase
---

###アプリ内課金とは
アプリ内課金とはiTunes Storeを通してアプリ内の様々なものを購入する機能です。  
例えば、  

1. 電子書籍アプリの購読  
2. コミュニケーションアプリのスタンプ  
3. ゲームアプリの協力なアイテム  

などが該当します。  
2と3に関しては買い切りプランとして、1は一定の購読期間を持ったプランとしてAppleから提供されています。  
本記事では一定の購読期間を持ったプラン **自動更新購読(Auto-Renewing subscription)** について書きます。

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

###自動更新購読(Auto-Renewing subscripion)とは
アプリ内課金をアプリに実装する場合、もちろん事前にできる限りのテストをしなければいけません。Appleはテスト用にSandbox環境を用意しています。  
では、Sandbox環境と本番環境は何が違うのでしょう。  
また、アプリがどの状態のときにSandbox環境で試すことができ、どこから本番環境となるのでしょう。 

####購読期間の違い
まず、Sandbox環境では、テストをしやすくするために実際の購読期間よりも短い期間となっています。  
1年を1時間と考えれば、1〜12ヶ月まで空で時間を計算できますが、7日のみ3分が割り当てられていることに注意しましょう。  
(同じ割合で行くと、流石に短すぎるからだと思われます。)  

Sandbox環境  

* 7日 → 3分  
* 1ヶ月 → 5分  
* 2ヶ月 → 10分  
* 3ヶ月 → 15分  
* 6ヶ月 → 30分  
* 12ヶ月 → 60分  

####Sandbox環境では自動更新キャンセルができない
Sandbox環境では、iTunes Connectで作成するテスト用のApple IDアカウントを使用します。  
しかしながら、本物のアカウントでないため、設定アプリから購読の自動更新をキャンセルすることはできません。  

![設定アプリから管理画面には遷移できない](/images/in-app-purchase-1.png)  

自動更新をキャンセルすることができない代わりに、自動で5回だけ購読が更新されます。  
つまり、1回購読を始めれば、計6回分だけ購読が続きます。  

しかしながら、自動で購読が更新されると言っても、即時更新ではありません。  
(恐らく、Apple側のサーバでもバッチを回しているのではないでしょうか...)  

例えば、Sandbox環境で1ヶ月プランを購入したとしましょう。  

* 1回目の購入日: 2016/09/25 10:00:00.000  
* 1回目の期限日: 2016/09/25 10:05:00.000
* 2回目の購入日: 2016/09/25 10:07:00.000  

このように「1回目の期限日」と「2回目の購入日」が完全一致するわけではないのです。  
こういった仕様を理解した上でテストを進めないと思わぬ誤った結果を出してしまうかもしれません。  

####請求の有無について
開発中に最も気になるのは請求の有無ではないでしょうか？  
心配しなくともSandbox環境では請求は発生しません。  
もちろん本番環境では請求が発生します。  

####アプリの状態別環境
iOSアプリは様々な状態が存在します。  
それは次の5つです。  
Develop / Adhoc / TestFlight / Promotion Code / Release  

各状態で本番環境とSandbox環境のどちらが使えるのかまとめると下記のようになります。  

* Sandbox環境  
  * Develop  
  * Adhoc  
  * TestFlight  
* 本番環境  
  * Promotion Code  
  * Release

Promotion Codeでアプリを配布するには、Appleの審査を通過する必要があります。  

###レシート検証
自動更新購読は買い切りと違って、継続した結果をサービス提供側のサーバで保持しておく必要があります。  
(サービス特性上、それがないのはサーバを使わずにアプリ内で完結する場合のみでしょう。)  
そこで避けては通れないのが **レシート検証** です。  

レシート検証には2種類の方法があります。  

* ローカル検証  
  * アプリ内で独自実装します。  
  * 独自実装する理由は公開されたサードパーティ製ライブラリを使うと仕組みがバレてしまうからです。  
* App Store検証  
  * 信頼できるサーバを用意してサーバサイド処理として実装します。  
  * アプリから直接App Storeに検証しに行くことは脆弱性があるため推奨されていません。  

####App Store検証
App Store検証できるように、Appleは **レシート検証API** を用意しています。  
ここでも本番環境とSandbox環境でアクセス先URLに違いがあります。  

* Sandbox環境URL: https://sandbox.itunes.apple.com/verifyReceipt  
* 本番環境URL: https://buy.itunes.apple.com/verifyReceipt  

共にリクエストパラメータは2種類になります。  

* `receipt_data`  
  * base64エンコードを施したレシートデータ  
* `password`  
  * iTunes Connectで発行した共有シークレットキー  

また、レスポンスデータも同じになります。  
ここでは重要なものだけ書きます。  

* `status`  
  * 有効なレシートの場合は **0** 。それ以外は幾つかの理由で無効なレシートとなっています。  
* `receipt`  
  * `bundle_id`: iOSアプリの`CFBundleIdentifier`  
  * `application_version`: iOSアプリの`CFBundleVersion`  
  * `in_app`: 購買履歴(自動更新購読の場合、更新された段階で追加されます。)  
  * `original_application_version`: iOSアプリの`CFBundleVersion`  
  * `product_id`: 購入アイテムのプロダクトID  
  * `transaction_id`: 購入アイテムのトランザクションID  
* `latest_receipt`  
  * base64エンコードを施した直近の更新に対応するトランザクションレシート  
* `latest_receipt_info`  
  * 購読履歴(自動更新購読の場合、更新された段階で追加されます。)  

上記のレスポンスデータ概要を見ると、  
`receipt.in_app`と`latest_receipt_info`は同じものに見えます。  
実際にはレシート検証APIを叩いたタイミングで取得できるデータは異なります。  
Appleは`in_app`を利用することを推奨しており、`latest_receipt`と`latest_receipt_info`は「 **iOS 6型のトランザクションレシートの場合のみ。** 」としています。  
しかしながら、iOS7以降でもこれらの値は返却されてきます。  
ここがネット上で調べてもはっきりしない部分ではあるのですが、筆者の試した感覚では`latest_receipt_info`は`in_app`よりもリアルタイムに情報が更新されている気がしています。  
`in_app`は自動で更新された後にレシート検証APIを叩いても1回目は必ず情報が更新されていません。  
1回目で返却された`latest_receipt`を`receipt_data`として利用すると正しく更新された`in_app`が返却されます。  
(Sandbox環境の場合であって、本番環境では検証できていません。)  

このことを考えると、`in_app`は非常に使いにくいものに思われますが、使い方に寄るのかもしれません...  
今後、`latest_receipt`と`latest_receipt_info`が廃止されてしまうかもしれませんし、そうなったときにAppleは事前に告知をするのか...などなど不安要素は実にたくさんあります。  

アプリ側では、起動時に課金周りの監視処理を必ず実行する必要があります。  
これを実行することで、自動更新購読の場合に更新を検知してサーバに送信するといったフローができるためでしょう。  

冒頭に上げた2種類のレシート検証APIの使い道は様々あるのかもしれませんが、  
筆者としては今のところ  

* ローカル検証  
  * 必ずアプリ側の処理に組み込む  
* App Store検証  
  * アプリ不具合やクラッシュなどで漏れたユーザへの対応  
  * アプリを起動しないユーザへの対応  

と考えています。  
つまりはどちらも必要なんですね。  

###まとめ
さて、筆者的にはもう少しローカル検証やリストアに関しての理解が乏しいと感じています。  
これからまだまだ関わる気もするので、また理解が進んだら続きを書こうかなと思います。  
と言ったところで本日はここまで。  

参考  

* [レシート検証 プログラミングガイド](https://developer.apple.com/jp/documentation/ValidateAppStoreReceipt.pdf)  
* [自動購買課金について](http://ameblo.jp/principia-ca/entry-12071724382.html)  
* [In-App Purchaseを実装する際に調べたことまとめ](http://qiita.com/monoqlo/items/24d36e3a95bc813a7276)  
* [iOSアプリの課金検証環境](http://qiita.com/koyopro/items/c6c387b136da1698f666)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
