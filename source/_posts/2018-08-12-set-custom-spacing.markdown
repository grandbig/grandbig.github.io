---
layout: post
title: "iOS11から使える非常に便利な『setCustomSpacing』"
date: 2018-08-12 23:06
comments: true
categories: ios swift UIStackView autolayout
---

### はじめに
`UIStackView` は様々なシーンで、 `AutoLayout` を飛躍的に使いやすくしてくれました。  
一方で、まだまだ改善の余地ありと感じさせられるところも多く、部分的に困るiOSエンジニアもいたのではないかと思います。  

例えば、  
『同一スペースを持つ複数Viewをレイアウトする』のには、  
 `UIStackView` が非常に有効であるものの、  
『一部異なるスペースを持った複数Viewをレイアウトする』には、  
あまり向いているとは言えない  
といったことなどです。  

今回はiOS11から上記例が解消されたことを紹介したいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### 題材の説明
まずは題材の説明です。  

* `UILabel` 3つを表示  
  * 上から `labelA` , `labelB` , `labelC` とする  
* 各 `UILabel` のスペースが異なる  
  * `labelA` と `labelB` 間のスペースは `50.0pt`  
  * `labelB` と `labelC` 間のスペースは `8.0pt`  

### iOS10まで
iOS10までは、多重に `UIStackView` を利用する必要がありました。  
それぞれに `spacing` を指定するしか方法がなかったからです。  

![多重にUIStackViewを生成](/images/set_custom_spacing_1.png)  

複雑な画面であればあるほど、 `UIStackView` が多重化し、 `xib` や `storyboard` の編集が重くなります...  

### iOS11以降
iOS11では1つの `UIStackView` でこの状況を打破することができます。  

構成はたったのこれだけで...  

![1つのUIStackView](/images/set_custom_spacing_2.png)  

あとはソースコードで `spacing` をカスタム化することで実現できます。  

```objective-c
// SampleViewController.swift

import Foundation
import UIKit

class SampleViewController: UIViewController {

    @IBOutlet private weak var stackView: UIStackView!
    @IBOutlet private weak var labelA: UILabel!
    @IBOutlet private weak var labelB: UILabel!
    @IBOutlet private weak var labelC: UILabel!

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.

        if #available(iOS 11.0, *) {
            stackView.setCustomSpacing(50.0, after: labelA) // ここが重要！！！
        } else {
            // iOS10.x以下は利用不能
        }
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }
}
```

### まとめ
さて如何でしたでしょうか？  
昨年までは、iOS10.xカスタマーも健在で、なかなか `setCustomSpacing` を利用するメリットが少なかったように思われます。  
しかし、今年はiOS12が登場しますし、自然とiOS11以上をサポート対象とするアプリも増えてくることでしょう。  
今年だからこそ `setCustomSpacing` は重要な改善の1つなのだと筆者は思います。  

と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
