---
layout: post
title: "UITextField 豆知識"
date: 2018-02-24 15:56
comments: true
categories: ios swift
---

### はじめに
最近、`UITextField` を何かとカスタマイズすることが多いので、忘れないように豆知識的に残しておこうと思います。  

アジェンダ  
① `UITextField` の色変更  
② `UITextField` のマージン変更  
③ `UITextField` の `Attribute` 対応  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### `UITextField`の色変更

`UITextField`を以下のように定義しているとします。  

```objective-c
@IBOutlet weak var textField: UITextField!
```

#### 入力した文字色の変更

以下のようにすれば、テキストカラーを赤色に変更できます。  
`textField.textColor = UIColor.red`

#### 文字入力時のキャレットの変更

以下のようにすれば、キャレット色を赤色に変更できます。  
`textField.tintColor = UIColor.red`

#### `Placeholder` の色の変更

以下のようにすれば、`Placeholder`の色を赤色に変更できます。  

```objective-c
textField.attributedPlaceholder = NSAttributedString(string: textField.placeholder ?? "", attributes: [NSAttributedStringKey.foregroundColor: UIColor.red])
```

### `UITextField`のマージン変更

```objective-c
let frame = CGRect(x: 0, y: 0, width: 16, height: 16)
textField.leftView = UIView(frame: frame)
textField.leftViewMode = .always
```

`textRect`, `editingRect`, `placeholderRect` を `override` する方法もありますが、  
上記であれば、1つの処理で入力後/入力中/Placeholder全てに適用されます。  

![UITextFieldにマージンを追加](/images/uitextfield_custom.png)  

### `Attribute` 対応

例えば、文字入力後にボタンを押下した後にバリデーションチェックをかけたいことがあるかもしれません。  
バリデーションチェックの結果、違反文字の色を変更する必要がある場合は以下のように `attributedText` に変更内容を反映させます。  

```objective-c
func setupAttributedString(textField: UITextField, attributeText: String, color: UIColor) {
  guard let text = textField.text else {
    return
  }

  let attributedString = NSMutableAttributedString(string: text)
  let colorRange = (text as NSString).range(of: attributeText)
  let font = UIFont.systemFont(ofSize: textField.font!.pointSize)
  let fontRange = NSRange(location: 0, length: text.count)
  attributedString.addAttribute(.foregroundColor, value: color, range: colorRange)
  attributedString.addAttribute(.font, value: font, range: fontRange)
  textField.attributedText = attributedString
}
```

![UITextFieldにAttributeを追加](/images/uitextfield_custom_2.png)  

### まとめ

意外に `UITextField` のカスタマイズってするんですよね〜  
今後も利用するシーンが多いと思うので覚えておかなきゃ。  
本日は以上。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
