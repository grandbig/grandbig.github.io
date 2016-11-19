---
layout: post
title: "Swiftでドキュメントコメントを正しく書こう！"
date: 2016-01-24 00:13
comments: true
categories: ios swift reference
---

####Swiftでのドキュメントコメントの書き方を覚えよう！
先日、[jazzy: リファレンス自動生成ツール](http://grandbig.github.io/blog/2016/01/12/jazzy/)の記事を書きました。  
その中で幾つかのドキュメントコメントの書き方を紹介しましたが、改めて説明しておきたいと思います。  

Swiftではドキュメントコメントの中でマークアップ言語の書式が有効に働きます。  
代表的なマークアップ言語の書式は下記の通りです。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####マークアップ言語の書式

#####見出し

```objective-c
# 最大見出し
## 大見出し
### 中見出し
#### 見出し
##### 小見出し
###### 最小見出し
```

![見出しを書いた場合](/images/documment_comment_1.png)  

#####リスト

```objective-c
- リスト項目１
    - リスト小項目１
    - リスト小項目２
    - リスト小項目３
- リスト項目２
- リスト項目３
```

![リストを書いた場合](/images/documment_comment_2.png)  

#####順序リスト

```objective-c
1. 順序リスト項目１
    1. 小順序リスト項目１
    2. 小順序リスト項目２
    3. 小順序リスト項目３
2. 順序リスト項目２
3. 順序リスト項目３
```

![順序リストを書いた場合](/images/documment_comment_3.png)  

#####ItalicとBold

```objective-c
normal

*Italic*

**Bold**
```

![ItalicとBold](/images/documment_comment_4.png)  

#####URLリンク

```objective-c
[URLリンク](https://developer.apple.com/jp/)
```

![URLリンク](/images/documment_comment_5.png)  

#####罫線

```objective-c
abcdefg
***
123456
		       
---
あいうえお
___
かきくけこ
```

![罫線](/images/documment_comment_6.png)  

#####コード表記
(Octopressの都合上、ここでは書き方が異なりますが、ご了承ください。)  

\` \` \`  
self.sampleMethod()  
\` \` \`  

![コード表記](/images/documment_comment_7.png)  

####Swiftのドキュメントコメントの書式

#####MARK

```objective-c
// MARK: - Properties
var sample:String = "sampleProperty"

// MARK: - Method
func sampleMethod() {
	print("This is sampleMethod.")
}
```

![MARK](/images/documment_comment_8.png)  

#####FIXME

```objective-c
func sampleMethod() {
	// FIXME: - ログ出力ではなく、アラートを出す
	print("This is sampleMethod.")
}
```

![FIXME](/images/documment_comment_9.png)  

#####TODO

```objective-c
func sampleMethod() {
	// TODO: - ログ出力ではなく、何か処理を書く(まだ決まっていないけど)
	print("This is sampleMethod.")
}
```

![TODO](/images/documment_comment_10.png)  

#####Parameter / Throws / Returns

```objective-c
/**
 サンプルメソッド
	      
 - parameter name: パラメータの書き方
 - throws: 例外処理の書き方
 - returns: 戻り値の書き方
*/
func sampleMethod2(name:String) throws -> String {
									         
	return "result: " + name
}
```

![Parameter / Throws / Returns](/images/documment_comment_11.png)  

#####Description内で利用可能な文言
この他にクラス, プロパティ, メソッドの説明(Description)を書く際に利用可能な予約語的なものが幾つかあります。  

```objective-c
/**
 - Attention: 注意書き
 - Author: 作成者が一人の場合
 - Authors: 作成者が複数人の場合
 - Bug: バグの詳細
 - Copyright: 著作権の所在
 - Date: 日付(作成日, 更新日など)
 - Experiment: 実験内容
 - Important: 重要事項
 - Invariant: 不変事項
 - Note: その他、必要事項
 - Precondition: 事前条件
 - Postcondition: 事後条件
 - Remark: 備考
 - Requires: 要求事項
 - SeeAlso: 参照事項
 - Since: いつから実装されているか
 - Version: バージョン
 - Warning: 警告				   
*/
```
上記のように書くことで、 **太字** かつ **各用語に属する説明** が付与されます。  

![予約語1](/images/documment_comment_12.png)  

![予約語2](/images/documment_comment_13.png)

####まとめ
さて、いかがだったでしょうか？  
これだけ知っていれば、ほぼ確実に困ることはないと思います。  
どこまで頑張って書くかは開発者次第かもしれませんが、やはり初めてコードを見る人でも極力伝わるようにしておくことは必要不可欠かと思います。  

といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
