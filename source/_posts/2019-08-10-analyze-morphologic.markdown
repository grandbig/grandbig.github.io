---
layout: post
title: "iOS形態素解析で見る幽白、海藤戦のタブー能力"
date: 2019-08-10 21:13
comments: true
categories: ios swift
---

### はじめに
最近、業務で形態素解析の話が出ました。  
形態素解析はサーバサイドで実行していることが多く、あまりアプリでは馴染みがないと思っていたのですが、  
意外とそんなことはなく、iOSでも簡単に試すことができることを知り、今更驚きました。  

因みに、筆者が形態素解析と聞いて真っ先に思い出したのは、  
「幽遊白書の海藤戦のタブー能力」でした笑。  

今回はiOSでの形態素解析を、海藤戦のタブー能力を織り交ぜつつ説明したいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### iOSで形態素解析
iOSでは、形態素解析のための標準APIとして `NSLinguisticTagger` クラスが用意されています。   
これを利用すると、解析は非常に簡単で下記のように数行で実行できてしまいます。  

```objective-c
// 解析対象の日本語文章
let text = "カウンターでワインを片手に人を待っています"

// 言語に日本語を指定
let tagSchemes = NSLinguisticTagger.availableTagSchemes(forLanguage: "ja")
let linguisticTagger = NSLinguisticTagger.init(tagSchemes: tagSchemes, options: 0)
linguisticTagger.string = text

// 解析を実施
linguisticTagger.enumerateTags(in: NSRange(location: 0, length: text.count), scheme: .tokenType, options: []) { (tag, tokenRange, sentenceRange, stop) in
  let word = (text as NSString).substring(with: tokenRange)
  print("\(word): \(String(describing: tag!.rawValue))")
}
```

上記の結果は、  

```objective-c
カウンター: Word
で: Word
ワイン: Word
を: Word
片手: Word
に: Word
人: Word
を: Word
待っ: Word
て: Word
い: Word
ます: Word
```

のようになりました。  
なかなか正確に単語ごとに分解できているように見えますね。  

因みに、全てひらがなの文章があった場合にはどうなるのかというと...  

```objective-c
// 例文
let text = "かうんたーでわいんをかたてにひとをまっています"
```

結果は、  

```objective-c
か: Word
うん: Word
た: Word
ー: Word
でわ: Word
いん: Word
を: Word
か: Word
たて: Word
に: Word
ひと: Word
を: Word
まっ: Word
て: Word
い: Word
ます: Word
```

のようになりました。  
なかなかに難易度が高いようですね...  

### 海藤戦
では、基本的な説明が終わったところで、いよいよ海藤戦の再現をしてみましょう。

まずは、先程の実装を元に、形態素解析をかけた日本語の文章が特定の単語を含むかどうか調べる処理を以下の通り作ります。  

```objective-c
private func analyzeMorphologic(_ text: String, target: String) -> Bool {
  var words = [String]()

  let tagSchemes = NSLinguisticTagger.availableTagSchemes(forLanguage: "ja")
  let linguisticTagger = NSLinguisticTagger.init(tagSchemes: tagSchemes, options: 0)
  linguisticTagger.string = text
  linguisticTagger.enumerateTags(in: NSRange(location: 0, length: text.count), scheme: .tokenType, options: []) { (tag, tokenRange, sentenceRange, stop) in
    let word = (text as NSString).substring(with: tokenRange)
    words.append(word)
  }
  return words.contains(target)
}
```

続いて、単純に文字列を含んでいるかどうか調べる処理も以下の通り作ります。  

```objective-c
private func contains(_ text: String, target: String) -> Bool {
  return text.contains(target)
}
```

この2つのメソッドに、  

```objective-c
text: ああついでに氷をいれてくれ
target: あつい
```

を引数として設定してみましょう。  

結果は、  

```objective-c
analyzeMorphologic: false
contains          : true
```

となりました。  

前者の `analyzeMorphologic` は形態素解析をすることで、『意味を持った単語が文章内に存在しているか』を見るのに対して、  
後者の `contains` は `target` となる文字列が『文章内に存在しているか』だけを見ています。  

よって、海藤のタブー能力が前者だと気を抜いていると、  
実は後者がルールであったがために引っかかり、魂を抜かれてしまうことになってしまうのです。  

### まとめ
海藤のタブー戦であれば、むしろ形態素解析をする必要はなかったのですが、  
現実にソフトウェアを実装する現場では形態素解析が重要な場面は多分にあることでしょう。  

と言ったところで、本日はここまで。  

### 参考URL

* [Apple Developer Document: NSLinguisticTagger - enumerateTags](https://developer.apple.com/documentation/foundation/nslinguistictagger/1410036-enumeratetags)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
