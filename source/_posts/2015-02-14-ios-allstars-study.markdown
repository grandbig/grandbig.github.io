---
layout: post
title: "iOS オールスターズ勉強会に参加しました！"
date: 2015-02-14 13:16
comments: true
categories: ios 勉強会
---

####iOS オールスターズ勉強会
本日、[iOSオールスターズ勉強会](http://eventdots.jp/event/311301)に参加してきました。  
iOSに特化した勉強会やイベントに参加するのは初めてでしたが、かなりいろいろと勉強になりました。  
そして、まだまだ自分はヒヨッコだと痛感させられました。  
まだまだ-1レベルの情弱プログラマーであることを再認識したところで、いつの日かこのブログを**0から始める情弱プログラミング**に変えてみせたいと思いつつ、勉強会で学んだことを報告したいと思います。  

勉強会のテーマは下記の通りです。  

* Adaptive Collection View
* Swift製ライブラリの良い書き方を考える
* let UIWebView as WKWebView
* 通信のパフォーマンス改善
* 効率的なアプリ開発のベストプラクティス
* WatchKit を実際にさわってみてわかったこと
* 長生きするために心臓に悪いリリースはもうやめよう
* エンジニア戦記 ~ 小さなチーム 大きな未来 ~
* まだiOSでリッチな演出に疲弊してるの？

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####Adaptiva Collection View
[本日の資料](https://speakerdeck.com/ishkawa/adaptive-collection-view)  
[石川さんのGitHub](https://github.com/ishkawa)(近々、大きめのライブラリを公開予定とのこと)  

下記はメモ  

* iOS8でViewControllerが進化した
* セルを並べてアイテムを表示する系のアプリではiPhone, iPadで実装を分けていた。
    * iPhoneはUITableViewを利用
    * UICollectionViewを利用
* UICollectionViewを使えば、iPhone, iPadで実装を分ける必要がなくなる
    * 実装はUICollectionViewFlowLayoutを使えば、割りと簡単にできる
    * UICollectionViewLayoutAttributeをUITableViewに似せる
    * [サンプルコード](https://github.com/ishkawa/sandbox)
* UICollectionViewを使うと、セルの計算がたいへんなのでは？
    * estimatedItemSizeを設定することで、Self Sizing Cellを利用できる(iOS8から)
    * NSLayoutConstraintを適切に配置するとセルの大きさを自動的に計算してくれる
    * iOS7はsystemLayoutSizeFittingSize:をitemSizeForIndexPath:で呼べばOK

因みに、この手法の優位性に気づいたのは、  
営業: 『急ぎでiPhoneだけで良いので、この機能を実装してください』  
↓  
UICollectionViewを使って開発  
↓  
営業: 『iPad版も実装してくれてありがとうございます！』
↓  
気づいたらiPad版もできていた  

まとめ  

* UITableViewはUICollectionViewである  
* iPhoneはiPadである  
* 同じコードは２度書かない  

【Q&A】  

Q: Cellの並び替え  
A: UITableViewは元々実装されている。UICollectionViewは自分で頑張って実装することになる。  
もしくは諦めてUITableViewを使う  

####Swiftで使いやすいAPIを考える
[本日の資料](https://speakerdeck.com/kishikawakatsumi/swiftderaiburariwoshu-kuji-falsepurakuteisu)  
[岸川さんGitHub](https://github.com/kishikawakatsumi)  

下記はメモ  

* Swift製ライブラリの良い書き方を考える  
    * 人に使われるコードを書くときに、どうすれば良いか  
    * enumが良いという話をすると長くなるので10分でできる話をする  

* Objective-Cとの違い  
    * データ型, Optional, オーバーロード, デフォルト引数, メソッドチェーン, エラー処理などの違いがある

* データ型
    * Class, Struct, Enum, Tuple, Function, Array, Dictionary, Set, Optionalなどがある
    * Optional型を指定すると、nilを許さない。コンパイル時にわかる(Objective-Cではnilは無視してできていた)
    * Optionalか否かは明確に設計した方が良い(Swiftは型に厳格な言語なので、明確にして使い方を固めた方が良い)
    * 外に公開しているメソッドは引数にOptionalはつけない(プライベートメソッドはつけても良い)

* オーバーロード
    * オーバーロードとは引数を変えた同じ名前のメソッドをたくさん用意すること
    * デフォルト引数は補完がイマイチ
    * ユーザがメソッドを書いたときに、先に引数の多いメソッドが補完で上がってくると良くない

* 戻り値のオーバーロード
    * 戻り値のオーバーロード(NSString, NSData型でそれぞれ返却する場合)はコンパイルエラーが出るので、`var let:NSData? = ....`と型指定をする
    * これはリズムが悪いので、メソッド名を分けることにした

* エラー処理
    * Objective-CならNSErrorのダブルポインタが一般的だけど、Swiftだと実装しにくい
    * SwiftではEither型を使う(Eitherをユーザが知らないと使えないけど)
    * 自分で定義した型でエラーを返すより、NSErrorを返したほうが良い

* playground
    * ドキュメントを書くのと一緒にplaygroundをつけてあげると、わかりやすくなって良い

* Functional style API
    * アドバンストモードで用意すると良い  
    * 標準的な書き方と分けて示すと良いと思う  

【Q&A】  

Q: Tupleで返すのは？  
A: EitherでなくてTupleで返すのもあり。  
Tupleで受けると、受けては使いづらい。  
検討した結果、やめた。  

####let UIWebView as WKWebView
[本日の資料](http://www.slideshare.net/taketo1024/let-ui-webviewaswkwebview)  

* Swiftについて
    * Swiftは数学と相性が良い  

* UIWebVIewとWKWebViewについて
    * UIWebViewはJavaScriptがSafariよりも遅い  
    * WKWebViewはSafari並に速い  
    * iOS7を切る訳にはいかない...  
    * 毎回、バージョンで条件分岐を書いて、処理を分けるのは美しくない...
    * 複雑な処理は見えないところに置く方が良い(UIWebViewとWKWebViewのぶん機処理を隠蔽せよ)
    * iOS7と気持よく別れられるように対応しておくと良い  
    * [去年のAdvent Calendar](http://techblog.yahoo.co.jp/ios/let-uiwebview-as-wkwebview/)にも載っている  

個人的には参加前から最も注目していた内容でした。  
WKWebViewは実装で使いたいと思っていたものの、現状、条件分岐が多くなりそうで使っていなかったのですが、これを機会に真似て実装したいと思いました。  

【Q&A】  

Q: UIWebViewとWKWebViewにおけるDelegateメソッドはどうする？  
A: Delegateはそのまま書いている。  
iOS7をサポートしなくなったときにUIWebViewのDelegateの部分だけ消すことを考えている  

Q: UIWebViewと揃えるのであればUIWebViewだけ使えば良いのでは？  
A: WKWebViewの良い所は積極的に使っていきたい(処理が速いところとか)  
iOS8ユーザには体験できる部分は体験してほしいと思っている。  

####通信のパフォーマンス改善
[本日の資料](http://www.slideshare.net/susieyy/ios-ios-44665266?ref=http://eventdots.jp/event/311301)  
[wantedlyさんのGItHub](https://github.com/wantedly)

下記メモ  

* そもそも通信のパフォーマンス改善に取り組んだ理由
    * 海外進出する上で通信品質の違いに耐えうるか調べる必要があった
    * 海外の通信品質が悪いので、それに耐えうるパフォーマンス改善に取り組んだ

* ボトルネックを調べた
    * New Relic Mobileでボトルネックを分析した。(大雑把なボトルネック解析には十分)
    * Pony Debuggerでもっと細かく分析できるものがあった  

* [Pony Debugger](https://github.com/square/PonyDebugger)の使い方
    * CocoaPodsでiOSアプリ側にいれる。  
    * サーバ側にもモジュールのインストールが必要。中身はPythonでできている  

* ボトルネック調査の結果
    * 画像にボトルネックがあった  
    * 画像の処理に関して[SDWebImage](https://github.com/rs/SDWebImage)を使っていた  
    * まずは、SDWebImageの仕組みから知る必要がある  
    * SDWebImageのバグを発見して、PullRequestを出したらマージされた  
    * [Facebookの発展途上国向けの改善施策](http://agilecatcloud.com/2014/12/02/facebook-%E3%81%8C%E8%AA%9E%E3%82%8B%E3%83%A2%E3%83%90%E3%82%A4%E3%83%AB%EF%BD%A5%E3%83%81%E3%83%A5%E3%83%BC%E3%83%8B%E3%83%B3%E3%82%B0%E3%81%AE%E6%A5%B5%E6%84%8F%EF%BC%9A%E3%81%93%E3%82%8C%E3%81%A7/)がネット上に載っていた  

* WebP	
    * WebPとはGoogle製のフォーマット(1/3程度のサイズに抑えられる)  
    * SDWebImageはWebPに対応している  
    * wantedlyではサーバはAWSを使っている  
    * WebPに変換する画像サーバを構成するのがたいへんだった  
    * [NginxでWebPに変換するものをGitHubにあげている](https://github.com/wantedly/nginx-image-server)  

* 具体的な施策
    * 通信帯域により取得する画像サイズを変更する  
    * 通信品質は『通信開始・終了の経過時間と受領画像サイズ』から推測  
    * キャッシュ時は計測しない  
    * 独自にキャッシュ上に取りに行こうとする画像のサイズよりも大きい物がないか確認するようにしている  
    * 実装の可否はデベロッパの通信帯域制限機能で改善結果を確認した  

* 他に考えられる対策
    * MessagePackでJSONを圧縮する  
    * SPDYで通信コネクションをまとめる  
    * 通信帯域で同時接続数を調整する  

####効率的なアプリ開発のベストプラクティス
[本日の資料](http://www.slideshare.net/yayugu/ss-44667286)  

* iOSの一大イベントがここ2年連続で実施されている
    * Conference with Developers

* 効率的な開発の必要性
    * メインでない機能を創るときはできる限り効率的に開発したい  

* どうすれば速く開発できるのか
    * やることを減らす
    * UI, 通信の実装コストを減らす  

* UIの実装コスト
    * 端末の性能向上, Auto Layout, Storyboard, Self Sizing Cell  
    * Self Sizing Cellが便利なので、ユーザはiOS8を使って欲しいのが本音
    * 改善の余地としては、React.jsみたいなアプローチ(Viewのdiff & patch)があると良さそう  

* 通信の実装コスト
    * 仕様に依存。サーバ側が決める。  
    * どちらでも行える処理であれば、サーバ側に処理を任せる。  

* API
    * 共通化、RESTにとらわれない
    * Mantle, JSONModelライブラリの都合に合わせた形で返して欲しい

* State
    * クライアント側でCacheを持たない  
    * ローカルストアを利用してサーバ側とのdiffがあれば取得しにいく処理はしない  

* サーバ側仕様変更による弊害  
    * キレイなRESTful API
        * 特化したAPIの作成は断られる。  
    * レガシーサーバ
        * 触るのが怖い
        * リファクタリングするとデグレそう

* サーバ側担当者の説得の仕方  
    * パフォーマンスの向上などメリットを説明
    * クライアント側の実装の大変さをうったえかける
    * サーバ側も自分で書く

* 間を取る方法として、中継サーバをたてることも考えられる
    * 効率的でないので、やりたくない
    * 中継サーバの処理はClient開発者が書く  
    * ドワンゴはスカラーでProxy Serverを作っていたらしい  

【Q&A】  

Q: サーバ側にどこまで処理を任せているかの線引はあるか  
A: ユーザ体験があがるか、重要な機能かの２つによる  
twitterのツイートはオフラインで見れる ⇒ 意味がある  
設定画面がオンラインでないと見れない ⇒ それで良いのでは？  

####WatchKitを実際にさわってみてわかったこと
[本日の資料](http://www.slideshare.net/t26v0748/watch-kit-20150214?ref=http://eventdots.jp/event/311301)  
[堤さんのGitHub](https://github.com/shu223)  

* 自己紹介
    * Over&Out その後: ブログを書いている  
    * iOS7-Sampler, iOS8-Samplerを作っている  
    * iOS9-Samplerのアイコンは作成済み

* WatchKitについて
    * WatchKitはクラスはたったの15個
    * Watch ExtensionはiPhone側で実行
    * Watch Appは基本的に表示だけ

* 動的なイメージの表示
    * タイマーを利用する
        * 転送か描画が遅い？ ⇒ 非推奨
    * キャッシュを使う
        * フレームごとの画像はキャッシュしないようにプログラミングガイドに書いてある  
    * Animated Imageを利用する
        * iOS5から追加されているメソッドが役に立つ  

* テキスト入力
    * presentTextInputControllerWith~を使えば良い  
        * テキスト入力のインターフェースを表示するメソッド  

* Smart Replies + 音声入力  
    * 第一引数にフレーズの履いてる, 第二引数に.Plainを渡す  

* 絵文字入力  
    * 第二引数: .allowEmojiを渡せば良い  
    * シミュレータでは提供されていない  

* カスタムUI
    * WatchKit AppではCore Graphics使えない  
    * Appleの公式を見ると、連番アニメーションだった。  

* カスタムフォント
    * iOSと同様の手順で可能

* インターフェースのオーバレイ
    * WKInterfaceObjectはUIViewのsubview的にインターフェース同士を重ねられない  
    * WKInterfaceGroupのsetBackgroundImage:を活用すればできる

* originの調整
    * WKInterfaceGroupを活用(ダミーオブジェクト入れて、グループ化)  

* 角丸、Watchの画面サイズ分け対応などなど

【Q&A】  

Q: 手軽にアニメーションするには？  
A: [Flipbook](https://github.com/frosty/Flipbook)というOSSがある  
デザイナーが作成したものをアプリに入れるなら、動画としてもらう。  
Flashを使っていた人と仕事をしていたときは連番画像を吐き出してもらっていた  

Q: Apple Watchで画像の表示について  
A: AnimatedImageのオブジェクトの生成はiOS側でやるので問題ない  
AnimatedImageをWatch側でレンダリングするのは重い？⇒シミュレータでは問題なかった  

####長生きするための心臓に悪いリリースはもうやめよう
[本日の資料](https://speakerdeck.com/tokorom/chang-sheng-kisurutamenixin-zang-nie-iririsuhamouyameyou)  

* アプリの申請について
    * バグが見つかって、月々の課金処理に失敗したとしたら....非常に怖い  
    * Internal Testersでの最終チェックすると良い

* テストの質
    * AppStoreに公開されるものと同じ / 違うアプリのテストなのかどうか  
    * Appleの審査の前後のテストなのかどうか  

* Internal Testers  
    * iTunes Connectの新機能
    * iTunes Connectにサブミットしたアプリをテスト可能
    * iOS8以降でしか使えない
    * 25人までしか使えない

* Internal Testers vs CI  
    * 両方使いましょう(比べるものではない)

* CIの自動化
    * [Travis CIでiOSアプリのリリース作業を自動化する](http://kishikawakatsumi.hatenablog.com/entry/20141022/1413963656)という記事が良い  

* 万が一の事故に対して... 
    * 後から見返せない情報をバックアップしておく
        * Build Details: バージョンが進むと過去のバージョンの情報が見れなくなる。スクショをとっておくと良い。  
    * 過去のバージョンのアプリをとっておく
        * iTunesで実際に公開されたアプリをipaで持っておく
        * それをバージョン毎に保管しておくと良い
        * コードを見たいのであれば、Gitで良いが、そうでないときもある。  

【Q&A】  

Q: Internal Testers使うと課金できるのか？  
A:実際に購入ができるわけではない。たぶんsandboxに繋がっていると思う。  
厳密に言うと、課金テストはプロモーションコードでテストするしかない。  

Q: 自分でリジェクトしてサブミットし直すと速い気がする  
A: 体感的に速く感じることがある。  

####エンジニア戦記 ~小さなチーム 大きな未来 ~
[本日の資料](http://www.slideshare.net/yukihirai52/ss-44668339?ref=http://eventdots.jp/event/311301)  

* クラスメソッド
    * Developers.IOを書いている。  
    * 月間平均80万PV  
    * ブログを書いている会社だと思われがち

* iOSエンジニアとWeb-API担当者間で発生した問題
    * APIの返却データにステータスコードが入っている(HTTPステータスコードとは別に)
        * HTTPステータスコードで返すべき  
        * Web APIのレスポンスに独自ステータスコードを持つべきではない  
        * アプリ側でsuccessなのに、エラーハンドリングをしなくてはならない。  
    * 1画面で叩くAPIが多い  
        * RESTful APIをしたいがために上記のようなことが生じる  
        * 複数回APIを実行することで、エラーが発生しやすくなる  

* iOSエンジニアに求められるもの
    * Web APIの知識は必須
    * 改善案を提案できる力が必要
    * [Web API The Good Parts](http://www.oreilly.co.jp/books/9784873116860/)を読もう

身に染みる話ばかりで考えさせられるテーマでした。  
筆者はクライアント担当としてネイティブとWebの経験があり、サーバ担当としてAPサーバ開発の経験があるので、どっちが良いとは言い難く...。  

####まだiOSでリッチな演出に疲弊しているの？
[本日の資料](https://speakerdeck.com/ryusukefuda/madaiosderitutinayan-chu-nipi-bi-siterufalse)  

* カヤック自社開発アプリ
    * FILTERS
    * ダンボッコキッチン

* リッチな演出で何を使う？
    * Core Animation: コードが大量になる。大量オブジェクトを動かすとたいへん
    * Unity: ネイティブの機能を使うのにプラグインを書くのでたいへん
    * Cocos2d: Unityと同じ
    * OSS: いいものが見つからない

* SpriteKitが良いのでは？
    * iOS7から追加された2Dゲーム用のフレームワーク
    * パーティクルがXcodeでプレビューしながら編集可能
    * UIKitと同様に記述できる
    * SpriteKitがゲーム以外で使えるのか? ⇒ 使える
    * SKNode, SKScene, SKViewを使う
    * GUIでパラメータを簡単にいじることで演出を変えられる(コードを書く必要がない)
    * パーティクル自体にアニメーションをつけることもできる
    * UIViewのタッチイベントが効くようにするために、SKViewを透明かつタッチイベントを無効化する  
    * [サンプルコード](https://github.com/ryusukefuda/SpriteKit-Demo)

* SpriteKitを使うメリット  
    * Appleが提供している安心感
    * 既存のコードを邪魔することなく導入可能
    * デザイナーも利用可能

SpriteKitとUIKitを組合せた演出はありであるという結論に至っていました。  

【Q&A】  

Q: SpriteKitを使った理由  
A: マップとかカメラとの組合せを考えると、SpriteKitを選択した。  
3Dの演出をする場合はUnity一択になる。  

Q: パーティクルは重くないか？  
A: 重い。シミュレータだとカクついたりする。  
表示する場所を絞る。(SKViewの大きさを小さくするなど)  

といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>


