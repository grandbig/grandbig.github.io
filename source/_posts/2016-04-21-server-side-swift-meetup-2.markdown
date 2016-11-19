---
layout: post
title: "Tokyo Server-Side Swift Meetup #2 に行ってきました！"
date: 2016-04-21 19:24
comments: true
categories: swift
---

###Tokyo Server-Side Swift Meetup #2
今日は久しぶりに勉強会に行ってきた話です。  
今日行ってきた勉強会は[Tokyo Server-Side Swift Meetup #2](http://tokyo-ss-swift.connpass.com/event/29240/)です。  
筆者はSwiftでサーバサイドを書けるようになるとAppleが発表してからというもの、Swift3.0の公開を心待ちにしていました。  
いざ、GitHubで公開された瞬間は何とも言えない感動を覚えました。  

時は流れ...筆者は未だにSwift3.0を全く触っていません！  
日々、世の中で公開されるサーバサイドSwift関連の情報も全然追いきれていません...  
これはマズイということでまずは形だけでも参加してみることにしました。

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

ということで勉強会のメモ書きです。  

###Swiftプロトコル指向なCのラッパーを作る

[資料のリンク](https://speakerdeck.com/yusukeito/swift-purotokoruzhi-xiang-nacfalseratupawozuo-ru)  

* CoreAudio→Swiftからたたくと問題があった  
  * APIはC言語
  * AVFoundation系を叩くことが多いので、直接使う機会はあまりないかも  
  * C言語でのサンプルを紹介  
  * C言語は辛い, 1箇所でも間違えると落ちる・辛い  
  * AudioObjectGetPropertyData, AudioObjectSetPropertyData  
* Swiftでは入力の方に対応する出力の型として表現できる(Protocolのassociated-type)  
* [HALAudio](http://github.com/novi/HALAudio)  
  * Core AudioはPrefixが特殊なのでわかりにくい  
  * C言語よりSwiftの方がエラーハンドリングが楽(try-catchを使えるから)  
* プロトコルの場合の制約
  * コンストラクタのある型(class, struct, enum)を一連の制約の中に入れることができない    
  * プロトコル自体に値は持たせられない  

###The Ecosystem of Web Development with Swift

[資料のリンク](https://speakerdeck.com/takebayashi/the-ecosystem-of-web-development-with-swift)  

* Web Applicationを開発する上で使うツール  
  * Web Application Framework  
  * Web Application Servers etc  
* FrameworkとServerの選び方  
  * Framework  
    * [PerfectLib](https://github.com/PerfectlySoft/Perfect)  
      * 2015/11に発表  
      * オーソドックスな形で使える  
      * PerfectServerModuleInitという名前は固定(癖がある)  
      * デプロイがしんどい  
    * [Kitura](https://github.com/IBM-Swift/Kitura)  
      * IBM製  
      * 2016/02に発表  
      * 至って普通の書き方  
      * PerfectLibよりもシンプル！  
      * Kitura-netでKitura Appはデプロイできる  
      * サーバとFrameworkが密になり過ぎている  
      * PerfectLibをKitura-netで使うことはできない
      * サーバとFrameworkは疎結合であるべき  
      * Python: WSGI, Perl: PSGI, Ruby: Rack  
        * サーバとFrameworkが疎結合で好きに組み合わせることができる  
    * [Nest](https://github.com/nestproject/Nest)  
      * WSGIとかのSwift版  
      * Swift Protocolの  
      * 非常にシンプル  
      * Nest.swift1個だけ  
      * [Swifton](https://github.com/necolt/Swifton), [Frank](https://github.com/nestproject/Frank)はNestをサポートしている  
      * [Curassow](https://github.com/kylef/Curassow/blob/master/docs/index.rst), [http4swift](https://github.com/takebayashi/http4swift)はNestをサポートしている  
        * http4swiftはオススメしない  
      * [Inquiline](https://github.com/nestproject/Inquiline)でちょっと楽に  
      * Applicationの部分を変更せずにサーバを変えられる  
      * Nest Tools: [NestTest](https://github.com/nestproject/NestTest), [NestTestSuite](https://github.com/nestproject/NestTestSuite)  
    * [Open-Swft](https://github.com/open-swift)  
      * Cross project standards for Swift  
      * 先週まではSwiftXという名前だった  
      * [C7(Component)](https://github.com/open-swift/C7), [S4(Server-Side)](https://github.com/open-swift/S4), [D5](https://github.com/open-swift/D5)  
        * C7: Data, Stream, URIなどの基本的なデータ構造を提供  
        * S4: Server-Side Swift StandardsなのでS4  
          * More Swifty  
          * Nestの競合(PSGIとかのSwift版)  
          * NestよりもSwiftっぽい  
          * 非同期/同期をIFレベルで対応  
          * S4はサーバ側も仕様が決まっている  
            * [Vapor](https://github.com/qutheory/vapor), [Zewo](https://github.com/Zewo/Zewo), [Slimane: Frameworks Supporting S4](https://github.com/noppoMan/Slimane)  
            * [VeniceX](https://github.com/VeniceX/Venice), [Skelton: Servers Supporting S4](https://github.com/noppoMan/Skelton)  
      * Slackチャンネルがある  
    * Nest vs S4  
      * Community: S4の方が活発, NestはCommunityが死んでる  
      * API: Nestは安定している, S4はAPIがころころ変わる  
      * Spec: Nestはシンプル(画面1個に収まる), S4は複雑(C7をベースにしているから)  
      * 開発時は好きな方を選んでOK  
      * Nestで実装されたものが多いが、S4はこれから良くなる  
      * 好きなFrameworkを選んで、それに対応した好きなサーバを選ぶ  
      * NestでApp, S4でサーバしたい場合はアダプタを自作しましょう  
    * PerfectLibは使わないで!  

###Up to Simane 0.2 and the Server Side Swift Standards

[資料のリンク](https://speakerdeck.com/noppoman/up-to-slimane-0-dot-2-and-the-server-side-swift-standards)  

* Slimane  
  * Slimane開発者  
  * Express inspireしたFramework  
  * 超軽量  
  * 完全に非同期  
  * 超高速  
  * モジュール化されています  
  * マルチプロセスで動く  
  * Swift3.0サポートしている(Swift2.2は動かない)  
  * Open-Swiftを採用している  
  * 常に最新を追いかける思想で開発します  
  * SUV: Swift Wrapper for libuv  
  * Skelton: Stream based tiny Asynchronous HTTP Server  
* 0.1から結構変更した  
  * 極力頑張ってExpressっぽく書けるようにしている  
* Open-Swift  
  * 開発がめちゃ速い(次の日はCompileが通らない...)  
  * forkして、そのバージョンを使うようにしている  
  * SwiftonがNestからOpen-Swiftに移行するかも？  
* Slimaneでの書き方  
  * めっちゃExpressっぽい  
  * RESTful APIの書き方とか...  
  * RouterもMiddlewareもExpressっぽい...  
  * Session  
    * Redisを使ったSession Storeの例を紹介  
  * Slimane.Static  
    * 今はバグっているので、すぐに直します！  
  * BodyParser  
    * application/json, application/x-www-form-urlencoding  
  * Response with Template Engine  
    * MustacheViewEngine  
  * Renderable Protocol  
    * ReactのView EngineをSwiftで作る夢  
    * 夢であって、現実には存在しない  
  * Cluster  
    * シングルスレッドをコア数分、展開する  
    * 書き方はNode.jsライク  
    * Node.jsのClusterモジュールを真似て作った  
  * IPC between the Master and the Workers  
  * Let it crash  
    * メモリが80%こえたらRestartするといったAppの作り方ができる  
    * プロセス管理ができる  
  * DataBase  
    * Slimaneには今はない  
    * MySql-Swiftがすごく良い  
      * SlimaneはNo Blocking I/Oだからダメ...  
  * Process.qwork  
    * Blocking I/OをNo Blocking I/Oに見せかける  
    * Synchronous I/O  
  * パフォーマンス  
    * ABテストをやってみた  
    * Slimaneが動くバージョンで試してみた  
    * Swiftのバージョンによって動かないFrameworkは多い  
    * Nginxは早過ぎる...  
    * Expressより相当Slimaneは速い(Swiftが速いことが証明された)  
    * REVELよりもVAPORの方が速い(Swiftが速いことを証明)  
  * Slimaneの今後  
    * Promise対応必須  
    * Stream Response  
    * MySqlをネイティブ非同期で通信  
    * Command line IF  
    * Test Codes  
    * Webフロントとの親和性を高めたい  

今日は以上です。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
