---
layout: post
title: "Sansan×SONYアプリ勉強会に参加しました"
date: 2016-02-16 19:26
comments: true
categories: ios android study
---

####Sansan×SONYアプリ勉強会
本日は勉強会への参加報告です。  
[Sansan×SONYアプリ勉強会](http://connpass.com/event/25411/)に参加してきました。  
なかなか参加したいアプリ関連の勉強会って少ないんですが、今回は面白そうだなと思って、結構、早めに申し込んでいました。  
当日見てみるとキャンセル待ちも数十名になっていたので、珍しく早めに申し込んで良かったなと思いました。  
では、早速、筆者が今回の勉強会で得た感想等々を書いていきます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####チームでのiOSアプリ開発
個人で開発するときはさして困らないものの、  
チームで開発すると途端に困る...それがコンフリクトであることは周知の事実ですが、  
それを如何にして減らしていけるのか  
そんな話を聞いてきました。  

[スライドシェアの資料](http://www.slideshare.net/ssuser65a265/ios-58353672?ref=http://connpass.com/event/25411/presentation/?utm_campaign=new_event_links_to_participant&utm_source=notifications&utm_medium=email&utm_content=detail_btn)  

**【コンフリクトが起こった原因】**  

* 開発スタイル
    * 命名規則, ハードタブとソフトタブが入り交じる, commitの粒度...は人によって異なる  
    * チーム内の基準を設ける必要がある(Cocoa向けコーディングガイドラインなど)  
    * GitHubとPullRequestで互いにレビューをし合うことで問題を早期発見  
* Storyboard  
    * 同時に画面を作ると、コンフリクトが起こるので、複数画面が同じStoryboardに入るのをやめる  
    * 1画面1つのStoryboardで作った
* Localizable.strings  
    * 日本語だとUTF-16なのできれいにマージできず、GitHubでdiffが見れない  
    * 同時に実装を進めるとコンフリクトが起こる  
    * 機能単位に分割して取り組むと良い
* project.pbxproj  
    * 大きい機能追加だと確実にコンフリクトが発生  
    * Editorでも開けないので、完全手直しになる...  
    * [mergepbx](https://github.com/simonwagner/mergepbx)を使った
        * α版だったので、適用後に必ず目を通すことにした
* 証明書 (Code Signing)  
    * Xcode上で『Fix Issue』ボタンのあるアラートが出たとき、『Fix Issue』を押すと困ることが起こる  
    * Appleのストレージから証明書をRevokeして秘密鍵が行方不明, CIサーバまで影響なんてことが...  
    * 『Fix Issue』は押さないようにチームで決めた  
    * チームで一人はProvisioning Profileや証明書のことがわかる人を入れる  

**【まとめ】**    

* Xcodeでの複数人開発には工夫が必要  
* マージにかかる労力を減らす→開発前に対策するのが理想  
* 『Fix Issue』は何も解決しないので押さない  

####家電と繋がるアプリの開発 - リモコン＆テレビ番組表アプリ "TV SideView" の事例
こちらはSONYが出している『TV SideView』アプリに関する話でした。  

**【目指していること】**  

* 新しい・優れたユーザ体験(UX)の追求  
* 理想のUX実現のために『家電, モバイル, ゲーム, アプリ』と横断して開発する  

**【家電とアプリの繋がり例】**  

* [Chrome Cast](https://www.google.co.jp/chrome/devices/chromecast/) / [Nexus Player](http://www.google.co.jp/intl/ja_jp/nexus/player/) / AirPlay  
* [Nest](https://nest.com/)  
* [Dropcam](https://nest.com/camera/meet-nest-cam/)  

**【家電とアプリが繋がることについて】**  

* 家電の操作について  
    * アプリの機能: リモコン画面をスマホに表示, アプリから番組を選択, 番組の視聴/予約  
    * 長所: 高度な操作指示が可能なUIを実装できる, ネットワーク・電波があればどこでも使える  
        * 高度な操作の例: 家の中の機器をまとめて操作することができる  
    * 短所: アプリ起動の手間, ブラインド操作できない, 操作フィードバックがない  
* アプリが家電のディスプレイになる  
    * 画面のない家電の画面にアプリがなれる  
    * 手元にない家電の操作ができる  
    * 画面のある家電であっても、セカンドディスプレイになれる  
* アプリならではの良さ
    * 番組の詳細情報の閲覧ができる
    * 人気番組を知る(予約数の人気ランキングで見せる)  
* スマホが家電のセンサーになる  
    * 近くの家電を見つける  
    * ユーザの接近を知る
    * ユーザ行動による家電の操作
    * 音声入力
    * 着信に応じて音量を下げる
* 家電をパーソナライズ
    * モバイル端末の履歴からおすすめの番組をレコメンド  

**【家電のUXの進化について】**  

* アプリと繋がることで家電のUXの進化を加速  
* アプリであれば機能追加やアップデートが容易なので、家電と組み合わせることでUXの進化が劇的に加速する  

**【家電とアプリを繋げることの大変さ】**  

* 予約操作はTCPで → 信頼性を重視  
* 機器検索やリモコンマウスカーソル機能はUDPで → パフォーマンスを重視
* 家電とクラウドサーバでは、可用性と反応速度が大きく異なる  
* ハード性能・コストの制約  
* 家電の状態変化に対応できるアプリに...(リモコンで急遽、OFFにされるとか)  
* 世界で10万台以上の家電がBotなどに感染されている...  

**【SONYでは】**  

* TV SideView 1400万DL達成  
* データドリブンによるアプリ改善(機能の利用頻度, アクセス数を解析)  
* 6週リリース体制(スクラム開発を導入)  

####アプリの不具合を少なくするために
[スライドシェアの資料](http://www.slideshare.net/kenichitatsuhama/ss-58316068)  

**【勉強になったこと】**  

* 外注では品質担保が難しいというのが通例なのかもしれない
* Android Studio Inspect Codeは良さそうなので使ってみたい  
* InferをInspect Codeと併用させるのはあり  
* 設定で「アクティビティを保持しない」にすることで想定通りの動作をするか見ることで、堅牢なアプリになる  
onCreateを通らない値は復帰しないと覚えておく(想定していた処理が呼ばれない, パラメータがないとかならないように...)  
* ViewPagerではFragmentは3つまでしか保持しないので注意  
* [Master of Fragment (Android Professional Developerシリーズ)](http://tatsu-zine.com/books/master-of-fragments)は良いらしい

**【お役立ちツール】**  

* [Robotium](https://github.com/RobotiumTech/robotium)  
* [Jacoco](http://eclemma.org/jacoco/)  

####Android / iOS アプリのコード共有 - J2ObjC を使ってみた -
J2ObjCは個人的に気になっていたので、話が聞けて良かったです。  

**【J2ObjCを使うきっかけ】**  

* 家電連携以上の価値を生み出したい  
* 開発効率を上げる事で目指す  
* Android開発に強い人が多い
* 開発の段階、段階で適用が可能

**【J2ObjCの特徴】**  

* Native UI非対応 → UIはPlatform Nativeで開発したほうが良いというスタンス  
* ビルドが複雑  
* あくまでも開発量・メンテナンスコストの削減になるのであり、単一コードになるわけではない

**【J2ObjCを使った結果のサンプル】**  

* 19%の開発量の削減ができた
* Bluetoothの接続・通信開始処理も実装できた
* JUnitのテストコードも変換された  

**【お役立ちツール】**  

* [j2objc-sample-reversi](https://github.com/tomball/j2objc-sample-reversi)  

今回、アプリ系の勉強会に久々に参加しましたが、  
自分自身まだまだ知らないこともたくさんあるので非常に勉強になりました。  
また、継続して、気になった勉強会には参加していきたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
