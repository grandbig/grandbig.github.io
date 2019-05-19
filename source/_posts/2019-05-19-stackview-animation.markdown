---
layout: post
title: "UIStackViewとConstraintで変わるアニメーション"
date: 2019-05-19 17:07
comments: true
categories: ios swift
---

### はじめに
今日は`UIStackView`とアニメーションに関するメモです。  
通常、`UIStackView`を利用すれば、ラッピングされた内部の`View`間の距離は、  
`Spacing`で指定することができます。  
逆に言えば、`NSLayoutCostraint`を指定する必要はありません。  

ですが、もしアニメーションで工夫を加えた場合はその限りではないという話をしたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### UIStackViewを利用したアニメーションサンプル１
まずは、今回実装するサンプルを説明します。  
最終Viewは、  

![サンプルView](/images/uistackview_animation_1.png)  

です。  

下記アニメーションに従って、この最終Viewになります。  

1. 赤色Viewが徐々に表示される  
2. 青色Viewが徐々に表示される  
3. 最終Viewになる  

`Storyboard`では下図のように実装しています。  

![Storyboardの設定](/images/uistackview_animation_2.png)  

`UIStackView`だけで、うまく実装できているかがわかります。  
アニメーションの実装は下図の通りです。  

```objective-c
import UIKit

class ViewController: UIViewController {

    @IBOutlet weak var redView: UIView!
    @IBOutlet weak var blueView: UIView!

    override func viewDidLoad() {
        super.viewDidLoad()

        UIView.animate(withDuration: 1.0, animations: {
            // 赤色Viewが徐々に表示される
            self.redView.alpha = 1
        }) { _ in
            UIView.animate(withDuration: 2.0, animations: {
                // 青色Viewが徐々に表示される
                self.blueView.isHidden = false
                self.blueView.alpha = 1
            })
        }
    }
}
```

この実装によって実現されるアニメーションは下記の通りです。  

![サンプル１のアニメーション](/images/uistackview_animation_4.gif)  

### UIStackViewを利用したアニメーションサンプル２
サンプル１では赤色Viewと青色Viewが中央から上下に離れる形で表示されたかと思います。  
もしこれを、赤色Viewについていく形で青色Viewを表示したい場合は工夫が必要になります。  

その工夫とは、`Storyboard`上で`UIStackView`にラッピングされた2つのViewに`NSLayoutCostraint`を与えてやります。  

![StoryboatdでNSLayoutConstraintの制約を追加](/images/uistackview_animation_3.png)  

これによりアニメーションは下記のようになりました。  

![サンプル２のアニメーション](/images/uistackview_animation_5.gif)  

### まとめ
手軽にアニメーションを実現しようとすると少々の工夫が必要なことがわかりました。  
本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
