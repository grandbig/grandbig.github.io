---
layout: post
title: "UITableViewで任意のスクロール位置に移動させる方法"
date: 2018-01-28 21:01
comments: true
categories: ios swift
---

### はじめに

最近、開発していて微妙にハマった `UITableView` を `reloadData` 後に、スクロール位置を移動させる方法についてメモしておきたいと思います。  
`UITableView` で一覧表示をするアプリを開発している際に、  
新規データに更新後、スクロール位置を変更したい場合があるかと思います。  
その方法について書いていきます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### スクロール位置を変更させる方法
ざっとググってみるだけで幾つか方法が見つかると思います。  

例えば...  

* `self.tableView.contentOffset = CGPoint(x: 0, y: -self.tableView.contentInset.top)` を設定する  
* `self.tableView.scrollRectToVisible(CGRect(x: 0, y: 0, width: 1, height: 1), animated: false)` を設定する  
* `self.tableView.scrollToRow(at: indexPath, at: UITableViewScrollPosition.top, animated: false)` を設定する  

これらを以下のように実行してみても想定通りにスクロール位置が動いてくれません。  

```objective-c
/// UITableViewのデータ更新処理
///
/// - Parameters:
///   - data: 更新したいデータ
func configure(data: [Int]) {
  // データ更新処理など実行 (今回は省略)
  self.tableView.reloadData()

  let indexPath = IndexPath(row: 0, section: 0)
  self.tableView.scrollToRow(at: indexPath, at: UITableViewScrollPosition.top, animated: false)
}
```

これは `reloadData()` が実行されて、描画が完了する前に次の `scrollToRow` が実行されているからだと思われます。  

これをどうやって解決すれば良いかと言うと...  
データ更新後の `UITableView` が描画された後にスクロール位置が変わるように、タイミングをずらしてやる必要があります。  
こんな感じで...  

```objective-c
/// UITableViewのデータ更新処理
///
/// - Parameters:
///   - data: 更新したいデータ
func configure(data: [Int]) {
  // データ更新処理など実行 (今回は省略)
  self.tableView.reloadData()

  DispatchQueue.main.async {
    let indexPath = IndexPath(row: 0, section: 0)
    self.tableView.scrollToRow(at: indexPath, at: UITableViewScrollPosition.top, animated: false)
  }
}
```

恐らく `reloadData` 実行指示の中にメインスレッドで実行する処理がキューとして溜められているはずです。  
その後に実行される `scrollToRow` を `DispatchQueue.main.async` 内で実行することで、メインスレッドのキューに処理を追加しています。  
こうすれば `reloadData` が終わった後に `UI` 処理を続けて実行できるというわけですね。  

### まとめ

今回の考え方は他の処理でも利用できるところがある気がするので覚えておいて損はないはず！  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
