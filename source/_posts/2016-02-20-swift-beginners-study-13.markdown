---
layout: post
title: "『Swiftビギナーズ勉強会 第１３回』に参加してきました"
date: 2016-02-20 13:51
comments: true
categories: ios swift study
---

####始めに
本日は[Swiftビギナーズ勉強会 第１３回](http://eventdots.jp/event/578498)に参加してきました。  
筆者はiPhoneアプリ開発自体は4年近く前から関わっているものの  
(初めはWebViewを使ったハイブリッドアプリから携わり、後にフルネイティブに...)、  
多くはObjective-Cで書いてきたため、Swiftでのコーディング経験は長くありません。  
そういった経緯や、iOS関連の勉強会を探していたこともあり、  
良い機会だと思って参加することにしました。  

今回の勉強会の概要はこちらの資料で説明頂きました。  
[概要資料](http://www.slideshare.net/YoshinoriKoba/swiftswift)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####ショートセッション

1. Swift × WordPress SwiftからWP REST APIを使ってWordPressの記事をリストする  
[発表資料](http://www.slideshare.net/YoshinoriKoba/swift-wordpressswift-wp-rest-apiwordpressswift-160220)  
    * メモ  
        * デバッグ方法に関するリンク  
[Xcodeでのデバッグ方法](http://blogios.stack3.net/archives/776)  
        * WP REST APIを使ったときにATSに関するエラーが発生した  
HTTPSサーバを用意するのはハードルがあるので、plistの設定で対応した  
    * 感想  
        * 個人的には[SwiftyJson](https://github.com/SwiftyJSON/SwiftyJSON)を使ったことがなく、やっぱり使った方が良いのかなと  
        * 最近オープンに公開されているAPIがHTTPだと、ATSに引っかかるので、世の中のオープンなAPIが載っているサーバはHTTPSになっていくのかなと  

2. mBaaSの基本的な扱い方の事始め  
    * 参考URL  
        * [ニフティクラウドmobile backendの事始めと導入手順まとめ](http://qiita.com/fumiyasac@github/items/53d9d45d880b0e523417)  
        * [ニフティクラウドmobile backendを利用した画像付きメモアプリ作成のポイントまとめ](http://qiita.com/fumiyasac@github/items/7549acb9610860de1733)  
    * メモ  
        * Coffre, 10秒虫食い算アプリ  
        * [Parse](https://parse.com/)とNCMBの比較  
        * Parseは **2017/01/29(日本時間)** でサービス終了するのでNCMBに乗り換えたい  
        * ParseサーバのソースはGitHubに公開されている  
        * Parseサーバのソースを見るとNode.js(Express)とMongoDBを利用している  
        * Frameworkが提供されているので、Referenceに従ってアプリ側にソースを実装していけば、自分でREST API叩くだとかしなくて良い模様  
        * FrameworkはObjective-C製なので、Bridge-Headerの作成が必要  
        * NCMBはParseと大差なさそう  
        * NCMBは日本語のコミュニティやフォーラムなどがあるので安心  
    * 感想  
        * いつも自分でNode.jsとMongoDBで構築しちゃうからmBaaS使ったことなかったな〜  

3. iOS9で楽になったAutoLayout  
[発表資料](https://speakerdeck.com/satotakeshi/shi-utukiyanai-ios9dele-ninatutaauto-layout)  
    * メモ  
        * iOS6で誕生したAutoLayout  
        * 徐々に進化して...iOS9でStackView, NSLayoutAnchorsが出た  
        * 入れ子で複雑なレイアウトをStackViewで実現可能  
        * これで等間隔にviewとか並べられるよ  
        * iOS8まではspacingのviewを作成して、それぞれAutoLayoutをつけていた...  
        * NSLayoutAnchorsが出たことでコードでも1行で制約を書ける(かなりソースコードが短縮される)  
        * NSLayoutConstraintはたいへんだった  
        * Visual Format Language → 複数の制約を一気につけれるが、不正な記法が判明するのが実行時...。記法が独特すぎる...  
    * 感想  
        * Dynamic Typeってあまり気にしたことなかった...  

4. Linterを使って快適なチーム開発をしよう!  
[発表資料](https://speakerdeck.com/akatsuki174/linterwoshi-tutekuai-shi-natimukai-fa-wosiyou)  
    * メモ
        * SwiftLintの話  
        * 『(』で引っかかる場合はautocorrect実行してやれば良い  
        * バージョンアップの頻度が高い  
    * 感想  
        * 先日、気になって取り組んだところだったので、復習になった  

####もくもく会
ショートセッションの後にもくもく会の時間がありました。  
あまりテーマを決めていなかったものの、1時間〜1時間30分程度の時間だったため、  
あまり大きなことはやらずに基本的なところから見直してみることにしました。  

そこで、Swift2.0から追加された新機能3つを試してみることにしました。  

* `do-catch`, `try`, `defer`  
* `guard`  
* `Protocol Extension`  

* 参考URL  
    * [Swift2.0で追加された新機能 3選](https://moneyforward.com/engineers_blog/2015/06/17/swift2/)  
    * [Swift2.0で追加されたguard statementを試してみる](http://llcc.hatenablog.com/entry/2015/06/13/235129)  

####まとめ
今回、初めてSwiftビギナーズ勉強会に参加させて頂きましたが、  
日本にはこんなにも幅広い年齢層、職種(専門プログラミングがSwift以外)の方々がSwiftに興味を示しているのだと改めて感じました。  
Objective-Cであれば、基本的なところは抑えられていると思えるようになってきましたが、  
Swiftに関してはまだまだ経験不足で学ぶべきところが多いと感じたのも事実です。  
だからこそやり甲斐があり、モチベーションが上がるんですけどね。  
といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
