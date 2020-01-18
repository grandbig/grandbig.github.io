---
layout: post
title: "SwiftでiOS&Androidアプリを開発！？〜Scadeチュートリアル〜"
date: 2020-01-19 01:07
comments: true
categories: ios swift
---

### はじめに
昨今、 `Flutter` や `React Native` などによるiOSアプリとAndroidアプリの同時作成が少しずつ現実的に実践されるようになってきており、  
筆者的には、2010年代前半以来のリブームのように感じられる今日此頃です。  

2010年代前半は、Facebookを筆頭に、最終的にはフルネイティブに舵を切り直すプロダクトが多かったイメージがあるのですが、  
各種OSの浸透および安定化に伴い、今回の流れはある程度続く可能性があるのではと思わずにはいられません。  

しかしながら、 `Flutter` は `Dart` というGoogle製の言語を利用し、  
`React Native` は `JavaScript` および `React` の知識が必要になります。  

 既に `Swift` でのiOSアプリの開発や `Kotlin` によるAndroidアプリの開発に慣れているエンジニアであれば、  
 言語書式が比較的似ていることから、OSやIDEの違いさえ把握できれば学習コストは大幅に抑えられる可能性があります。  

 一方で、それでは両OSアプリの同時作成の恩恵に預かることができないため、  
 何か良いものがないかな〜と思っていたところ、  
『 [Scadeは、Swiftを使用してAndroidアプリ開発を可能にすることを目指す](https://www.infoq.com/jp/news/2019/08/scade-swift-android-development/)』という記事を見つけました。  

これは面白そうだなということで、今回は[Scade](https://www.scade.io/)について見てみたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### Scadeとは
公式ホームページによると、 `Scade` とは『次世代のモバイルアプリの開発基盤』と言われています。  

具体的にできることとしては、  

* `Swift` 言語でiOS&Androidの両OSのアプリを開発できる  
* `SCADE Simulator` という `Scade` アプリ内で提供されているシミュレータが利用できる  
* `Xcode` や `Android Studio` で利用している各々専用のシミュレータも利用できる  
* GUI上でアプリをデザインすることができる  

などがあります。  

Swift5にてABIの安定化を達成したこともあってか `Scade` 自体のバージョンも `v1.0` に到達しています。  

しかしながら、現在、最新の `Scade` は **Xcode11** ( つまり **Swift5.0** )までしか対応しておらず、  
お使いのMacで `Xcode11.3` が入っている場合は `Scade IDE` 上でのコンパイルに失敗してしまうため注意が必要です。  

### チュートリアル - Hello World
何はともあれ、 `Scade IDE` を使ってチュートリアルを進めてみましょう。  
`Scade` のドキュメントは非常に充実しており、チュートリアルとして `Hello World` が用意されています。  

参考：[Scade: チュートリアル - Hello World](https://docs.scade.io/docshttps://docs.scade.io/docs)  

上記リンク先に動画付きで説明がなされているので、大筋で困ることはないと思います。  
ただ、ドキュメントが古いのか、全く同じように作成することはできなかったため、紹介がてら使い方を見ていきたいと思います。  

まず、ソフトウェアのDLページですが、[こちら](https://www.scade.io/download/)になります。  
※無料で利用できます。  

#### プロジェクトの作成
続いて、プロジェクトの作成方法を説明します。  

① ナビゲーションバーから、Scade Projectを選択する  

![Scadeプロジェクトを作成する](/images/scade_1.png)  

② プロジェクト名を決める  

![プロジェクト名を決める](/images/scade_2.png)  

これだけで、IDE上に②で指定した名前のプロジェクト名が作成されていることを確認できると思います。  

![IDE上に作成されたプロジェクト](/images/scade_3.png)  

#### レイアウトの作成
`Scade` の特徴でも説明したように、 `Xcode` や `Android Studio` 同様にGUIからレイアウトを決めることが可能になっています。  
今回は、『チュートリアルのHello World』ということで下図のようなレイアウトを作成します。  

![チュートリアルHello Worldのレイアウト](/images/scade_4.png)    

では説明していきましょう。  

① main.pageファイルを開く  

![main.pageファイルを開く](/images/scade_5.png)  

② 右メニューのPaletteのWidgetsからLabelとButtonをドラッグ＆ドロップする  

![右メニューのPaletteのWidgetsからLabelとButtonをドラッグ＆ドロップする](/images/scade_6.png)  

③ 右メニューのPaletteのLayoutsからGrid1つとVertical2つをドラッグ＆ドロップする  

![右メニューのPaletteのLayoutsからGrid1つとVertical2つをドラッグ＆ドロップする](/images/scade_7.png)  

④ Grid内にVerticalを2つ子要素として追加し、それぞれのVerticalの子要素としてLabelとButtonを追加する  

![Grid内にVerticalを2つ子要素として追加し、それぞれのVerticalの子要素としてLabelとButtonを追加する](/images/scade_8.png)  

⑤ Gridの上下左右にレイアウトを設定する  

![Gridの上下左右にレイアウトを設定する ](/images/scade_9.png)  

⑥ Label側のVerticalのレイアウトを設定する  

![Label側のVerticalのレイアウトを設定する](/images/scade_10.png)  

⑦ Button側のVerticalのレイアウトを設定する  

![Button側のVerticalのレイアウトを設定する](/images/scade_11.png)  

⑧ Labelのレイアウトと色を設定する  

![Labelのレイアウトと色を設定する](/images/scade_12.png)  

⑨ Buttonのレイアウトと色を設定する  

![Buttonのレイアウトと色を設定する](/images/scade_13.png)  

#### 実行処理の実装
今回のチュートリアルでは、下記機能を持たせます。  

* ボタンをタップしたときにラベルの文字列を変更する  

これを実現するために、 `main.page.swift` ファイルを開き、実行処理を書きます。  

![main.page.swiftファイルに実行処理を書く](/images/scade_14.png)  

```objective-c
// main.page.swift
import ScadeKit

class MainPageAdapter: SCDLatticePageAdapter {

	// page adapter initialization
	override func load(_ path: String) {		
		super.load(path)

    // ①
		let button1 = self.page!.getWidgetByName("button1") as! SCDWidgetsButton

    // ②
		button1.onClick.append(SCDWidgetsEventHandler{ _ in
      // ③
			let label =  self.page!.getWidgetByName("label1") as! SCDWidgetsLabel
			label.text = "Swift on Android rocks"
		})
	}
}
```

処理内容を説明すると、下記の通りです。  

① `getWidgetByName` を使ってボタン要素を取得しています  
② `onClick` でボタンのタップを補足し、 `append` で処理を追加しています  
③ ラベルの文字列変更のために `getWidgetByName` で取得したラベルの `text` に変更したい文字列を代入しています  

#### シミュレータでの確認方法
左上からターゲットとなるプロジェクトを選択すると、実行するシミュレータを選択できます。  

![シミュレータでの確認方法](/images/scade_15.png)  

シミュレータを選択した後で、三角の実行ボタンを押下することでシミュレータを実行できます。  

![実行したシミュレータ](/images/scade_16.png)  

因みに、下図のように簡単にシミュレータの端末を変更する機能も備わっています。  

![シミュレータの端末を変更](/images/scade_17.png)  

### まとめ
さて、如何でしたでしょうか？  
SwiftでiOS/Androidアプリを同時に開発できるなんて夢のような企画ですよね！  

ただ、

* Swiftバージョンへの追従ができていない
* `Scade` のIDEが `Eclipse` ベースでもっさりしている  
* `Xcode` と同じ名称の部品があるので、同じように使えると思うと案外うまくいかない  

といった課題も感じました。  

しかしながら、 `Scade` が実用レベルに到達するようになれば、 `Flutter` や `React Native` と同じく1つの選択肢として普及するようになる可能性もあることでしょう...  

まあ、今回は単純に発想が面白かったということで紹介にとどめたいと思います。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
