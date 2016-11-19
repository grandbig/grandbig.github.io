---
layout: post
title: "SwiftLintを使ってみよう！"
date: 2016-02-13 21:17
comments: true
categories: ios swift
---

####SwiftLintを使ってコード規約に則ったチーム開発を目指そう！
本日は[SwiftLint](https://github.com/realm/SwiftLint)を紹介したいと思います。  

SwiftLintとは  
チーム内で1つのコード規約に則って開発することを補佐してくれる静的解析ツールです。  

JavaScriptであれば[JSLint](http://www.jslint.com/)や[JSHint](http://jshint.com/)(最近であれば[ESLint](http://eslint.org/)でしょうか)ですし、  
Objective-Cであれば[OCLint](http://oclint.org/)が主流かと思います。  
それのSwift版ですね。  

筆者はあまり大規模なチーム開発を経験したことがないので、  
SwiftLintを使ったことがありません。  
なので、逆に新鮮な心持ちで使い方や良いなと思う点について紹介できればと思っています。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####SwiftLintの導入方法
早速、SwiftLintを導入していきましょう。  

1．homebrewでインストール  
下記コマンドでSwiftLintをインストールできます。  
`brew install swiftlint`

2．SwiftLintを適用したいXcodeプロジェクトの設定を変更  
下図のように **Run Script** を追加しましょう。  
![Run Scriptの追加](/images/swiftlint-1.png)  

続いて、下記のShell Scriptを記載しましょう。  

```objective-c
if which swiftlint >/dev/null; then
	swiftlint
else
	echo "SwiftLint does not exist, download from https://github.com/realm/SwiftLint"
fi
```

![Shell Scriptを記載](/images/swiftlint-2.png)  

たったこれだけで準備完了です。  
非常に簡単ですね！  

####SwiftLintを使ってみよう
導入ができたので、SwiftLintを使っていきたいと思います。  
因みに、今回導入したSwiftLintはバージョン **0.8.0** です。  

実際にBuildしてみるとわかるのですが、  
デフォルトで設定されているチェック内容では厳しすぎて気が遠くなります...。  
特にCocoaPodsでオープンソースのライブラリをプロジェクトに追加している場合、  
おびただしい数のErrorとWarningが出ます。  

![OSSライブラリもひっかかる](/images/swiftlint-3.png)  

よって設定をカスタム化したいと思います。  
カスタム化は非常に簡単です。  

xcodeprojファイルと同階層に`.swiftlint.yml`ファイルを作成します。  
このファイルに設定可能なものは下記になります。  

* `disabled_rules`  
デフォルトで有効となっているルールの中から無効にしたいルールを設定  
* `opt_in_rules`  
デフォルトで無効となっているルールの中から有効にしたいルールを設定  
* `whitelist_rules`  
他の2つとの併用はできず、これに記載したルールのみ適用したい場合に設定  
* `included`  
SwiftLintの対象に含めるファイルのパスを記載  
* `excluded`  
SwiftLintの対象から外すファイルのパスを記載  

`.swiftlint.yml`を作成する前のデフォルト設定は下図の通りです。  
`swiftlint rules`とコマンドを打てば確認できます。  

![SwiftLintのデフォルト設定](/images/swiftlint-4.png)  

因みに、SwiftLintのコマンドは下記です。  

```objective-c
Available commands:

	autocorrect   Automatically correct warnings and errors
	help          Display general or command-specific help
	lint          Print lint warnings and errors (default command)
	rules         Display the list of rules and their identifiers
	version       Display the current version of SwiftLint
```

筆者が作成した`.swiftlint.yml`ファイルは下記になります。  

```objective-c
disabled_rules:
# コードの記載のない改行を許可したい
- trailing_whitespace
# ネスト1つで怒られるのでDisableにしたい
- nesting

# 追加したOSSライブラリは対象に含めたくない
excluded:
- Pods/
- Podfile
- Podfile.lock

# 1行あたりの文字数制限を300に変更
# プロジェクト作成時にデフォルトで追加されるコメントをひっかけないため
line_length: 300

# 変数名の長さのMAXとMINを変更
variable_name:
  max_length:
    warning: 60
    error: 80
  min_length:
    warning: 1
    error: 0
```

実際に使ってみた結果、  
チームとして開発するには非常に良いのではないかと思いました。  
ただ、デフォルトの設定項目では全てをカバーし切れないことがあるように感じました。  
なので、自前ルールを作成する機会がそれなりにある気がしました。  

[SwiftLintのREADME](https://github.com/realm/SwiftLint)にも記載がある通り、  
SwiftLintでは自前でルールを作成することができます。  

といったところで本日はここまで。  

参考:  

* [SwiftLintの.swiftlint.yml記述方式とその詳細](http://qiita.com/akatsuki174/items/45205e0f0948f11e9db5)  
* [SwiftLint 0.8.0 アップデート内容まとめ](http://qiita.com/akatsuki174/items/c37dfcae3ff3a79fb8cf)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
