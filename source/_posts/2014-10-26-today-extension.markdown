---
layout: post
title: "Today Extensionのサンプルを作ってみよう！"
date: 2014-10-26 00:11
comments: true
categories: ios ios8 Extension
---

###Today Extension
本日は、iOS8から追加されたToday Extensionを使ったアプリを作ってみたいと思います。  
さて、早速説明していきましょう。  

####Today Extensionをプロジェクトに追加
まず、XcodeでSingle View Applicationを用いてプロジェクトを作成してください。  
これは通常のプロジェクトを作る過程と何ら変わりません(いつもの手順です。)  
次に**App Extensionを追加**しなければなりません。  
手順は下記です。  

1. Show the Project navigator > General > + ボタンを選択  
2. iOS > Application Extension > Today Extensionを選択  
3. 名前をつけて作成(ここでは**TodayExtensionActivity**とします。)  

<!--more-->

![+ボタンをクリック](/images/today_extension_1.png)  
![Today Extensionを選択](/images/today_extension_2.png)  

ここまで終われば、Runでシミュレータ上で表示してみてください。  
通知センターを表示すると、**Hello World**と表示されることがわかるはずです。  
![Hello World](/images/today_extension_3.png)  

####余白の削除
しかし、表示が不格好ですね...。  
これはデフォルトで余白が設定されているためです。  
文字を表示するだけなら、余白は不要なので、余白を消してみましょう。  
これはソースを追加することで実現できます。  

先ほど追加したTodayExtensionActivityのTodayExtensionViewController.mを編集します。  
下記のソースをTodayExtensionViewController.mファイルに追加しましょう。  

```objective-c
// TodayExtensionViewController.m
- (UIEdgeInsets)widgetMarginInsetsForProposedMarginInsets:(UIEdgeInsets)defaultMarginInsets
{
	return UIEdgeInsetsZero;
}
```

これにより、下記のように表示が変わることが確認できます。  
![余白を削除](/images/today_extension_4.png)  

####高さを調整
横方向の余白がなくなったので、違和感がなくなったものの、高さはもう少し広くしたいということがあるかもしれません。  
その場合は、ソースで高さを指定しましょう。  
TodayExtensionViewController.mのviewDidLoadメソッド内で高さを指定します。  

```objective-c
- (void)viewDidLoad {
	[super viewDidLoad];
	
	// 高さ変更
	self.preferredContentSize = CGSizeMake(0, 100);
}
```

これにより、下記のような表示になると思います。  
![高さを調整](/images/today_extension_5.png)  

####通知センターに表示されたWidgetタップでアプリを起動
せっかく通知センターに表示したのですから、アクションを追加したいですよね？  
ここでは、タップでアプリを起動させてみたいと思います。  
まずは、URLスキームを設定しましょう。  

1. Show the Project navigator > TARGETS > info > URL Typesを選択  
2. identifier, URL schemesを設定  
![URL schemeの設定](/images/today_extension_6.png)  

次に、TodayExtension側のStoryboardを開きましょう。  
Viewに**Tap Gesture Recognizer**をドラッグ＆ドロップで追加しましょう。  
Today View ControllerにTap Gesture Recognizerの項目が追加されるので、それを右クリックしてSent Actionsを設定しましょう。  
![Tap Gesture Recognizerを追加](/images/today_extension_7.png)  

今回は、Sent Actionとして、TodayViewController.mにtapActionメソッドを追加しました。  

```objective-c
- (IBAction)tapAction:(id)sender {
	NSURL *URL = [NSURL URLWithString:@"textension://"]; // カスタムURLスキームを指定
	[self.extensionContext openURL:URL completionHandler:nil];
}
```

ここでは、URLスキームとして**textension**と設定したため、上記のソースのように指定できます。  
上記設定後にシミュレータを起動して、通知センターのWidgetをタップしてみてください。  
アプリが起動されるはずです。  

いかがだったでしょうか？  

少しずつ、Today Extension機能を実装したアプリが増えてきています。  
これは開発者にとってもユーザにとっても非常に便利な機能だと思いますので、これから自作アプリには積極的に実装していきたいと思います。  

ということで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
