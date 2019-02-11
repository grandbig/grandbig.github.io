---
layout: post
title: "初心者だけしゃない！『iOSアプリ設計パターン入門』へのススメ"
date: 2019-02-11 15:28
comments: true
categories: ios architecture
---

### はじめに
今回は、[iOSアプリ設計パターン入門](https://peaks.cc/iOS_architecture)という書籍を紹介したいと思います。  

<div class="peaks_widget" style="overflow:hidden; padding:20px; border:2px solid #ccc;"><div class="peaks_widget__image" style="float:left; margin-right:15px; line-height:0;"><a target="_blank" id="purchase" href="https://peaks.cc/grand_big/iOS_architecture"><img alt="iOSアプリ設計パターン入門" style="border:none; max-width:140px;" src="https://s3-ap-northeast-1.amazonaws.com/peaks-images/ios_architecture_project_cover_alpha.png"></a></div><div class="peaks_widget__info"><p style="margin:0 0 3px 0; font-size:110%; font-weight:bold;"><a target="_blank" id="purchase" href="http://peaks.cc/grand_big/iOS_architecture">iOSアプリ設計パターン入門</a></p><ul style="margin:0; padding:0;">
<li style="font-size:90%; list-style:none;"><span>著者：</span>
<span>関 義隆,</span><span>史 翔新,</span><span>田中 賢治,</span><span>松館 大輝,</span><span>鈴木 大貴,</span><span>杉上 洋平,</span><span>加藤 寛人,</span></li>
<li style="font-size:90%; list-style:none;">製本版,電子版</li>
<li style="font-size:90%; list-style:none;"><a target="_blank" id="purchase" style="text-decoration:underline; color:#1DA1F2;" href="http://peaks.cc/grand_big/iOS_architecture">PEAKSで購入する</a></li></ul></div></div>  


こちらの書籍は[PEAKS](https://peaks.cc/)というクラウドファンディングの技術書版サービス内で成立したプロジェクトから生み出されたものです。  

以前、[Android アプリ設計パターン入門](https://peaks.cc/books/architecture_patterns)という書籍が執筆されていたのですが、  
個人的には『iOS版もあれば絶対売れるのに...』と思っていました。  
その後、ふとPEAKSのサイトを見た時に、待ちに待った `iOS版` が執筆のための応援を募っていることに気づき、その場で応援を即決したことを昨日のように覚えています。  

そしてリリースされた書籍を手に取り読み込んでいくうちに、  
予想通りの良書であったと再認識しました。  
そんな想いを少しでも共有したいと思い、本記事を書いていこうと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### 対象読者について
PEAKSのページを見ると、対象読者は下記のように記されています。  

```
◆ アプリをどんなアーキテクチャで作るか迷っている方
◆ ViewController, ViewModelがすぐに肥大してしまう方
◆ 今後もメンテナンスし続けるアプリを作る方
◆ アーキテクチャ変遷の歴史を知りたい方
◆ そもそも設計って何？という方
```

書籍のタイトルに『入門』というワードが入っているため、初心者向けのように思われるかもしれませんが、  
上記の対象者を見て頂くとわかる通り、所謂 `Tech Lead` 人材にとっても役立つ書籍と言えるかと思います。  

### 内容について
では、実際にどのような内容が書かれているのでしょうか。  
まずは目次から見ていきましょう。  

#### 目次

```
第Ⅰ部：設計を知る
 ├── 第１章：設計するということ
 ├── 第２章：設計にパターンを適用する前に
 ├── 第３章：Swiftらしく設計する
 └── 第４章：アーキテクチャのパターンを鳥瞰する
第Ⅱ部：iOSアプリのための設計パターン
 ├── 第５章：MVC
 ├── 第６章：MVP
 ├── 第７章：MVVM
 ├── 第８章：Flux
 ├── 第９章：Redux
 ├── 第１０章：Clean Architecture
 ├── 第１１章：アプリの起動経路 - Application Coordinator の導入
 ├── 第１２章：画面遷移のパターン - Router の導入
 └── 第１３章：第2部まとめ - アーキテクチャの選定基準
 第Ⅲ部：設計をサービスに導入する
 ├── 第１４章：Flux の導入例
 └── 第１５章：Redux の導入例 - 大規模アプリケーションに Redux を導入する

 ※細部は省略
```

続いて筆者が思う各部それぞれのポイントについて紹介します。  

#### 第Ⅰ部：設計を知る
第Ⅰ部は、  

1. 「設計とは何か？/なぜ設計をするのか？」の説明  
2. 設計をする上で知っておくべき&身につけておくべき知識  
3. 設計原則をSwiftでどう実現させるのか  
4. アーキテクチャの変遷(iOSならではの話を含む)  

などについて語られています。  
1や2, 4に関しては、iOSに閉じない話も含まれているため、アーキテクチャ自体の読み物としても面白いと思います。  

3で記載した通り、具体的なコードベースで設計原理のSwiftでの実現手法が書かれているため、  
設計本でありがちな思想の提示に留まらないところに本書籍の価値があると感じました。  

要は、『 **言いたいことはわかるけど、どう書けば良いの？** 』という読者の問いに答えられる書籍になっているということです。  

#### 第Ⅱ部：iOSアプリのための設計パターン
第Ⅱ部は、  

1. 各種設計パターンの説明と具体例の提示  
2. 画面遷移の複雑さをどう解決するかの紹介  
3. アーキテクチャを選定する難しさ  

などについて語られています。  

1にある通り、概念の図式はもちろんのこと、具体的にコードを用いて説明されているため、  
どんなアーキテクチャがあるのか知らないという方でも理解が進むように感じられます。  

「書籍で読まずとも、ネットでググれば大抵理解できるのでは？」と思われる方もいるかもしれません。  
その意見に対する見解としては、  
『 **各種アーキテクチャのスタンダードな理解/共通理解を持つための拠り所としての価値** 』  
だと思っています。  

つまり、  
ネット上には、各種アーキテクチャを説明する記事が多数見つかりますが、  
物によってはオレオレ要素が含まれているため、  
記事によって差異が見られ、『何が正しいの？』と思ってしまうことがあるということです。  

また、チーム開発をする際に、人によって信じる依代が異なると、  
コミュニケーションコストが増大する可能性もあります。  

この書籍が必ずしもアーキテクチャ選定時の拠り所にすべきとは言いませんが、  
1つの選択肢として与えられることは一読者として非常にありがたいと感じています。  

2に関しては、ネイティブアプリ特有の課題に突っ込んだ内容となっています。  
プロジェクトアサイン時に、キャッチアップのためソースコードを読むと思うのですが、  
画面遷移処理がキレイにまとめられているとそれだけで立ち上がりコストがぐっと楽になると思います。  

また、各種アーキテクチャでアプリ構築を始めたところ、  
「画面遷移処理はどうしよう？」と思うこともあったりするので、ありがたいナレッジの共有だと感じました。  

そして非常に共感したのが、3のアーキテクチャ選定に関する話です。  
『 **モダン開発に対する魅力がアーキテクチャの必要可否に勝って選定されることが本当に正しいのか？** 』
ということを考えるのに良い内容だと感じています。  

個人的にも、「流行りこそ正義」みたいな考え方が先行して失敗してしまった苦い思い出があるので、ぜひ読んで頂きたい章です。  
(ホント、なかなか難しいんですよね...。プロダクトやチームにとってベターなアーキテクチャ選定って...)  

#### 第Ⅲ部：設計をサービスに導入する
第Ⅲ部は、  

1. `Flux` や `Redux` を実開発で利用するための追加説明  
2. 開発だけでなく、テストに目を向けた説明  

などについて語られています。  

入門系の技術書籍を読んだ後に思いがちな  
『基本はわかったけど、応用に生かせない』というもどかしさ  
を読者が埋めやすくするようなサポートだと感じています。  

`Flux` と `Redux` の2パターンに限った話ではありますが、  
エラーハンドリング/テスト/DIなどの汎用的な内容も含まれているため、  
一読の価値があると思います。  

### まとめ
さて如何でしたでしょうか？  
個人的には、iOS関連の書籍に持続可能性を持たせることは非常に難しいと思っていたため、  
本書に出会った時はとても感動しました。  
今この時だけでなく、長期に渡って一助となる可能性を感じることができました。  
これから繰り返し読み込んで理解を深めたいと思いますし、実業務に活かせるポイントを多分に含んでいると感じています。  

とは言え、百聞は一見に如かずということで、実際に一読を考えてみてはいかがでしょうか？  

と言ったところで本日はここまで。  

<div class="peaks_widget" style="overflow:hidden; padding:20px; border:2px solid #ccc;"><div class="peaks_widget__image" style="float:left; margin-right:15px; line-height:0;"><a target="_blank" id="purchase" href="https://peaks.cc/grand_big/iOS_architecture"><img alt="iOSアプリ設計パターン入門" style="border:none; max-width:140px;" src="https://s3-ap-northeast-1.amazonaws.com/peaks-images/ios_architecture_project_cover_alpha.png"></a></div><div class="peaks_widget__info"><p style="margin:0 0 3px 0; font-size:110%; font-weight:bold;"><a target="_blank" id="purchase" href="http://peaks.cc/grand_big/iOS_architecture">iOSアプリ設計パターン入門</a></p><ul style="margin:0; padding:0;">
<li style="font-size:90%; list-style:none;"><span>著者：</span>
<span>関 義隆,</span><span>史 翔新,</span><span>田中 賢治,</span><span>松館 大輝,</span><span>鈴木 大貴,</span><span>杉上 洋平,</span><span>加藤 寛人,</span></li>
<li style="font-size:90%; list-style:none;">製本版,電子版</li>
<li style="font-size:90%; list-style:none;"><a target="_blank" id="purchase" style="text-decoration:underline; color:#1DA1F2;" href="http://peaks.cc/grand_big/iOS_architecture">PEAKSで購入する</a></li></ul></div></div>  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
