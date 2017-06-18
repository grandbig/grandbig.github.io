---
layout: post
title: "iOSでヘッダーを設定する3つの方法"
date: 2017-06-17 14:27
comments: true
categories: ios swift
---

### はじめに
今回は今更ではありますが、iOSでヘッダーを作るための方法について書いていきたいと思います。  

- 代表的な方法  
  - `NavigationController`を利用する方法  
  - `UIView` + `UINavigationBar`を利用する方法  
  - `UINavigationBar`の高さをカスタマイズする方法  

1つずつ見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### NavigationControllerを利用する方法
最も少ない手間でヘッダーを作るなら`NavigationController`ではないでしょうか。  
幾つかメリット/デメリットを上げてみます。  

- メリット  
  - 最も簡単にヘッダーを作成できる(Editor > Embed In > Navigation Controller)  
  - 画面遷移の設定が簡単(戻るも自動で設定される)  
  - ステータスバー(最上部の縦20pxの領域)を考慮する必要なし  
- デメリット  
  - ヘッダーの不要な画面に遷移するときにヘッダーの非表示をコードで書く必要がある  
  - 画面遷移時のアニメーション変更に手間がかかる  

### UIView + UINavigationBarを利用する方法
Storyboardを利用するなら初めに上げた「`NavigationController`を利用する方法」か「`UIView` + `UINavigationBar`を利用する方法」を使うと良いでしょう。  
わざわざ`UIView`を利用する理由は、ただ単に`UINavigationBar`のみを配置すると、ステータスバー領域に邪魔されて表示が崩れてしまうためです。  
これを回避するためだけに`UIView`を配置するという方法です。  

具体的には下図のような配置になります。  

![UIView + UINavigationBarの例](/images/ios_header_1.png)  

- メリット  
  - Storyboard上でキレイに見える方法でヘッダーを作成できる  
  - 画面遷移先でヘッダーが不要であれば配置しなければOK  
  - 画面遷移時のアニメーションのデフォルト選択肢が多い  
- デメリット  
  - 画面遷移先でヘッダーが必要な場合は毎回`UIView`を配置する必要がある(色の透過の考慮など面倒な側面あり)  

### UINavigationBarの高さをカスタマイズする方法
割りと昔からある方法です。  
ただし、昔(iOS6以前)は純粋の`UINavigationBar`の高さを変更したいという用途で使われていました。  
今回は通常の`UINavigationBar`ではステータスバーと被ってしまうため、ステータスバー分の高さを拡張したいという用途で利用します。  

高さをステータスバー分、拡張した`CustomNavigationBar`は下記のように作成できます。  

```objective-c
// CustomNavigationBar.swift
import Foundation
import UIKit

class CustomNavigationBar: UINavigationBar {
  override func layoutSubviews() {
    super.layoutSubviews()
    super.frame = CGRect(x: 0, y: 0, width: super.frame.size.width, height: 64)
  }
}
```

これをStoryboardで利用するのをオススメしないのは他のパーツと組み合わせて利用する際に、`AutoLayout`を利用して(客観的に見たら謎の)マージンを設定しないとならないためです。  

![CustomeNavigationBarをStoryboardで設定](/images/ios_header_2.png)  
![20のマージンを設定しないといけない](/images/ios_header_3.png)  

一応、メリット/デメリットも書いておきます。  

- メリット  
  - 「`UIView` + `UINavigationBar`を利用する方法」よりもソースコードがスマート(機能拡張して利用しているという意味で。)  
  - (ソースコードで書いて実装することを前提にするなら)自由度が最も大きい  
- デメリット  
  - Storyboardで利用するとキレイではない(画面ごとに謎マージンを設定しなくてはならない)  

### まとめ
さて如何でしたでしょうか？  
たまに必要性を感じる内容であることもあって、一度まとめてみようと思い、書いてみました。  
今の世の中なら、用途にあったOSSライブラリもたくさんあると思うので手法は3つには限らないでしょうね。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
