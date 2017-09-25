---
layout: post
title: "Clean Swiftを勉強してみよう！(1)"
date: 2017-09-25 23:42
comments: true
categories: ios swift
---

### はじめに
本日は[Clean Swift](https://clean-swift.com/)について書いていきたいと思います。  

#### Clean Swiftとは
Clean Swiftは簡単に言うと『[Clean Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html)のSwift版』です。    
参考までにClean Architectureの有名な図を掲載します。  
![Clean Architecture](/images/clean-swift_1.jpg)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

Clean Swiftアーキテクチャを採用することで受けられる恩恵として下記が考えられます。  

* 各種コンポーネントの責務を細分化することで、Massive ViewControllerの解消に繋がる  
* データの方向性が一方向になるため、各種コンポーネントの相互依存性が減り、TDD開発が進めやすくなる  
* 各種コンポーネントの責務がはっきりしているため、チーム開発する際に、実装が平準化される  

#### コンポーネントの関係性
各種コンポーネントの関係性を表した個人的に最もわかりやすいと感じた全体像が下図になります。  
[swifting.io](https://swifting.io/blog/2016/09/07/architecture-wars-a-new-hope/)から抜粋させて頂きました。  
![Clean Swift Architectureの図](/images/clean-swift_2.png)  

この関係性を説明するにあたって、各種コンポーネントの責務を理解しておく必要があるのでそれぞれ見ていきましょう。  

#### View
特に他のアーキテクチャと大きく違わない認識です。  

**責務：**  
・iOSアプリの見た目を表現する  

#### ViewController
`Massive ViewController` と揶揄される部分ですが、Clean Swiftでの責務は以下になります。  

**責務：**  
・ `Interactor` に具体的な処理内容を問い合わせる  
・ `Presenter` からの指示を受けて `View` を描画する  
・ `Router` に画面遷移を依頼する  

#### Interactor
`ViewController` から依頼を受け、 `Interactor` は下記を実施する責務を持っています。  

**責務：**  
・ `Worker` と `Presenter` を仲介する  
・ どんな条件で、 `Worker` に何の処理を依頼するのかハンドリングする  
・ `Worker` 経由で取得したレスポンスを `Presenter` に渡す  

#### Worker
`Interactor` から受けた依頼を実行します。  

**責務：**  
・ `API` 処理や `Core Data` / `Realm` などのアプリ内ローカルデータの処理をハンドリングする  
・ 取得データを `Model.Response` 形式で返却する  
・ 成功/失敗レスポンスをハンドリングする  

#### Presenter
`Interactor` から `Worker` 経由で取得したレスポンスを受け取った後に、 `Presenter` は下記を実行することを責務とします。  

**責務：**  
・ 受け取ったレスポンスを元に最適な表示(成功/失敗などの表示)になるようハンドリングする  
・ 受け取ったレスポンスを `Model.ViewModel` 形式に変換する  
・ `ViewController` に `Model.ViewModel` を渡して描画を依頼する  

#### Model
Clean Swiftアーキテクチャの肝といっても過言ではないのが `Model` です。  

**責務：**  
・ 各種コンポーネントを切り離し、各種コンポーネント間のやり取りに利用される  
・ `Request` / `Response` / `ViewModel` の3つの構造体を持つ  

**3つの構造体の説明：**  
・ `Request`  
　　・ ユーザの操作をInputパラメータとして内包したデータ形式  
　　・ `ViewController` から `Interactor` に渡される  
・ `Response`  
　　・ `Worker` 処理結果を内包しているデータ形式  
　　・ `Interactor` から `Presenter` に渡される  
・ `ViewModel`  
　　・ `ViewController` での描画に即したデータ形式  
　　・ `Presenter` から `ViewController` に渡される  


### まとめ


参考URL:  

* [Clean Swift公式ページ](https://clean-swift.com/clean-swift-ios-architecture/)  
* [Introducing Clean Swift Architecture (VIP)](https://hackernoon.com/introducing-clean-swift-architecture-vip-770a639ad7bf)  
* [swifting.io: #24 Architecture Wars – A New Hope](https://swifting.io/blog/2016/09/07/architecture-wars-a-new-hope/)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
