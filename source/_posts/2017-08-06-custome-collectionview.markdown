---
layout: post
title: "UICollectionViewCellをカスタム化しよう！"
date: 2017-08-06 03:41
comments: true
categories: ios swift
---

### はじめに
今回は基礎中の基礎ではあるものの、結構忘れがちなカスタム化についてメモ書きしておきたいと思います。  
その題材として `UICollectionViewCell` を使ってみます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### カスタムファイルの作成
まずは `xib` ファイルを作成します。  
今回は下記のように `UIImageView` を持たせるようにカスタム化させます。  
(`CustomCollectionViewCell.xib`とします。)  

![xibファイル](/images/custom-collection-view-1.png)  

これと対となる`swift`ファイルを作成します。  
(`CustomCollectionViewCell.swift`とします。)  

```objective-c
// CustomCollectionViewCell.swift
import UIKit

class CustomCollectionViewCell: UICollectionViewCell {

  @IBOutlet weak var imageView: UIImageView!

  override init(frame: CGRect) {
    super.init(frame: frame)
    self.xibViewSet()
  }

  required init?(coder aDecoder: NSCoder) {
    super.init(coder: aDecoder)!
    self.xibViewSet()
  }

  internal func xibViewSet() {
    if let view = Bundle.main.loadNibNamed("CustomCollectionViewCell", owner: self, options: nil)?.first as? UIView {
      view.frame = self.bounds
      self.addSubview(view)
    }
  }
}
```

この`xib`と`swift`ファイルを繋ぐために `xib`ファイルの`File's Owner`の`Custom Class`の`Class`にクラス名を入力します。  

![xibとswiftの接続](/images/custom-collection-view-2.png)  

### Storyboardにカスタム部品を配置
続いて、先程作成したカスタム部品を`Storyboard`に配置します。  
今回は `UICollectionViewCell` をカスタム化しているので、右メニューから `UICollectionView` をドラッグ&ドロップして持ってきます。  

![UICollectionViewをドラッグ&ドロップ](/images/custom-collection-view-3.png)  

持ってきた部品とカスタム化クラスを結びつけます。  
`右メニュー > Show the Identity inspector > Custom Class > Class` にクラス名を入力します。  

![カスタムクラスへの接続](/images/custom-collection-view-4.png)  

### CustomCollectionViewCellの表示
ここまでくれば後はいつも通り`UICollectionView`を使えば良いだけです。  

・ `Storyboard` 上で `Collection Reusable View` の `Identifier` に値を設定  
・ 下記の通りソースコードを実装  

```objective-c
import Foundation
import UIKit

class CreateShopMemoViewController: UIViewController, UICollectionViewDataSource {

  /// UICollectionView
  @IBOutlet weak var collectionView: UICollectionView!

  override func viewDidLoad() {
    super.viewDidLoad()
    self.collectionView.dataSource = self
  }

  override func didReceiveMemoryWarning() {
    super.didReceiveMemoryWarning()
    // Dispose of any resources that can be recreated.
  }

  // MARK: - UICollectionViewDataSource
  func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
    let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "CustomCell", for: indexPath) as? CustomCollectionViewCell   
    // 画像を設定 (今回はサンプルのためNoImageIconというものがあることを想定しています)
    cell?.imageView.image = UIImage(named: "NoImageIcon")

    return cell!
  }

  func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {  
    return 1
  }
}
```

その結果は下記の通りです。  

![CustomCollectionViewCellの表示](/images/custom-collection-view-6.png)  

因みに、今回のように、Viewを1枚ペタッと貼るだけであれば、  

```objective-c
// MARK: - UICollectionViewDataSource
func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
  let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "CustomCell", for: indexPath)   
  // 画像を設定 (今回はサンプルのためNoImageIconというものがあることを想定しています)
  cell.backgroundView = UIImageView(image: UIImage(named: "NoImageIcon"))

  return cell
}
```

とすれば良いだけです。  

### まとめ
今回は完全なるメモ書きでしたが、カスタム化の基礎なので、十二分に慣れておかないとですね。  
と言ったところで本日はここまで。  

参考  

* [カスタムViewをNibから初期化し、IBDesignableとIBInspectableで便利に使う](http://himaratsu.hatenablog.com/entry/ios/customview)  
* [xib 化した UITableViewCell を使うときの Tips](http://qiita.com/taketomato/items/7bf3f1dc2690c76079fb)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
