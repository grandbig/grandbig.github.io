---
layout: post
title: "Swiftの基礎を理解しよう！(1)"
date: 2015-05-16 23:14
comments: true
categories: swift ios
---

####Swiftを基本から覚えよう
さて、本日はSwiftについて基本から覚えていきたいと思います。  
筆者はこれまでObjective-Cを中心に開発を続けてきました。  
が、昨年Swiftが初めて発表されて以来、少しずつSwiftに関わることが増えてきました。  
正直なところ、Xcodeのバージョンアップごとに『 **割りと大きな変更が生じるSwift** を商用アプリに使うかどうか』は迷います。deprecatedになるメソッドはあるにしても『 **大幅な変更の生じないObjective-C** 』の方が安全ですし、何より多彩なライブラリが存在しています。  

とは言え、今後、Swiftしか使えないiOSアプリエンジニアが当たり前に存在していくことでしょう。  
そんなときにObjective-CもSwiftも当然のように使えるエンジニアとなっていたいので勉強しておこうと思う次第です。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####Swiftでプロジェクトを生成
言語でSwiftを選択してXcodeのプロジェクトを新規作成すると、ViewController.swiftは下記のように生成されます。

```objective-c
import UIKit

class ViewController: UIViewController {
	
	override func viewDidLoad() {
		super.viewDidLoad()
		// Do any additional setup after loading the view, typically from a nib.
	}

	override func didReceiveMemoryWarning() {
		super.didReceiveMemoryWarning()
		// Dispose of any resources that can be recreated.
	}
}
```

これまでのObjective-Cでは下記のように生成されていました。  

```objective-c
#import "ViewController.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];
	// Do any additional setup after loading the view, typically from a nib.
}

- (void)didReceiveMemoryWarning {
	[super didReceiveMemoryWarning];
	// Dispose of any resources that can be recreated.
}

@end

```

少し違和感があるかもしれませんが、ぱっと見た感じ、あまり変わらないようにも見えます。  
因みに、Objective-Cの場合はViewController.m, ViewController.hの2つのファイルが作成されますが、Swiftの場合は **swift** という拡張子のファイルのみです。  

####変数/定数の定義
Swiftでの変数/定数の定義の仕方を見て行きましょう。  

```objective-c
class ViewController: UIViewController {
	
	override func viewDidLoad() {
		super.viewDidLoad()

		var variableString:String		// String型の変数を定義
		let invariableString:String		// String型の定数を定義
	}
	<省略>
}
```

【ポイント】

* **var 変数名:変数の型**
* **let 定数名:定数の型**
* 1行の終わりは **改行** or **;(セミコロン)**

letは **定数** なので、

```objective-c
let invariableString:String		// String型の定数を定義

invariableString = "これは定数です。"
invariableString = "定数なので、2回目以降の値設定はエラーが発生します。"

```

のようなことを書くと、`invariableString`に対する2回目の値設定行で、  
`immutable value 'invariableString' may only be initialized once`エラーが発生します。  
(`var`は定数なので問題ありません。)  

####グローバル変数/定数の定義
Swiftのグローバル変数/定数の仕方を見て行きましょう。  

```objective-c
import UIKit

var globalVariableInt:Int = 3
let globalInvariableInt:Int = 5

class ViewController: UIViewController {
	<省略>
}
```

【ポイント】

* クラス外に定義した変数/定数はグローバル変数/定数
* グローバル変数/定数は共に初期値が必要

2つ目のポイントに書いた通り、初期値を与えないと`Global 'var' declaration requires an initializer expression or getter/setter specifier`, `Global 'let' declaration requires an initializer expression`といったエラーが発生します。  

####オプショナル型と非オプショナル型
オプショナル型とは、Swiftで唯一『 **nilの代入が許可された** 』型です。逆に『 **nilの代入を許可されていない** 』型を非オプショナル型と言います。  
では、Swiftのオプショナル型/非オプショナル型の仕方を見て行きましょう。  

```objective-c
class ViewController: UIViewController {
	
	override func viewDidLoad() {
		super.viewDidLoad()

		var variableString:String?		// String型&オプショナル型の変数を定義
		let invariableString:String?	// String型&オプショナル型の定数を定義
	}
	<省略>
}

```

【ポイント】

* var 変数名:変数型 **?** でオプショナル型
* let 定数名:定数型 **?** でオプショナル型
* オプショナル型の初期値は **nil**
* 非オプショナル型の初期値は何も初期値が入っていない(nilが入っているわけではない)

これまで特に意識せずに変数/定数を定義してきたと思いますが、『 **?** 』をつけていない変数/定数は自動的に非オプショナル型となります。  
よって、

```objective-c

var variableString:String
let inVariableString:String

variableString = nil
inVariableString = nil

```

とすると`Cannnot assign a value of type 'nil' to a value of type 'String'`とエラーが発生します。  

因みに、 **?** をつけた書き方は省略した書き方です。  
`var variableString:String?`は`var variableString:Optional<String>`と同じです。  

####オプショナル型の使い方(アンラップの仕方)
オプショナル型はそのままでは使えません。  
例えば、`var variableString:String?`は`String`型の変数を`Optional`(オプショナル)型の箱に入れているイメージです。  
なので、`String`型として値を扱うためには『 **Unwrap** (アンラップ)』する必要があるのです。  
(箱から`String`型を取り出すイメージです。)  

アンラップには幾つか方法があります。  

* Forced Unwrapping  
強制的にアンラップする方法。変数/定数名の末尾に **!** をつける

```objective-c

var fuga:String? = "FugaFuga"
println(fuga!.lowercaseString)

```

上記例で`fuga.lowercaseString`とすると`Value of optional type 'String' not unwrapped; did you mean to use '!' or '?'?`という静的解析でエラーが発生します。  
因みに、『 **!** 』をつけた場合、中身がnilだと実行時にエラーが発生するので注意。  

* Optional Chaining  
Optionalのメソッドやプロパティ参照時にnilチェックする方法です。  
nilでなければUnwrapしてnilであった場合はnilを返す方法です。  
実はUnwrapして返されるものもOptional型であったりします。  
ただし、そのメソッドやプロパティを使うことができるのです。  

```objective-c
var hoge:String?
var fuga:String? = hoge?.lowercaseString
println(fuga)

hoge = "HogeHoge"
fuga = hoge?lowercaseString
println(fuga)
```

上記を実行してみると、1回目のprintlnで`nil`が出力され、2回目のprintlnで`Optional("hogehoge")`と出力されます。  

* Optional Binding  
nilチェックをして、nilでない場合に限り、Unwrapして元の方を返す方法です。  
別名の変数/定数に値を入れ直して、以後、それを使っていく場合に使います。  

```objective-c
var hoge:String?

if var fuga:String = hoge {
	println("値が格納されていました。")
} else {
	println("nilでした。")
}

hoge = "HogeHoge"

if var fuga:String = hoge {
	println("値が格納されていました。")
} else {
	println("nilでした。")
}

```

上記の例文を実行すると、1回目は`true`, 2回目は`false`の条件分岐を通過するはずです。  

* Implicitly Unwrapped Optional  
使用時に自動的にUnwrapされて元の型が返される方法です。  
`nil`が入っているものに対して実行した場合はエラーが発生します。(静的解析時には判別されないことに注意)  

```objective-c

vat hoge:String! = "HogeHoge"
println(hoge.lowercaseString)

var fuga:String!
println(fuga.lowercaseString)

```

上記の例文を実行すると、1回目のログは出力されますが、2回目のログ出力時にエラーが発生します。  
因みに、『 **型!** 』は『 **ImplicitlyUnwrappedOptional<型名>** 』の省略形です。  

```objective-c

var hoge:String! = "HogeHoge"
var fuga:ImplicitlyUnwrappedOptional<String> = "FugaFuga"

```

こんな感じですね。  

####オプショナル型の意義
前章でオプショナル型と非オプショナル型が存在することがわかりました。  
では、このオプショナル型/非オプショナル型は何のために使うのでしょうか？  

【オプショナル型/非オプショナル型を使う理由】

* 静的解析でエラーを発見するため
* 実行時の想定外のエラーを防ぐため
* オープンソースとしてライブラリを作成した際に、ライブラリ利用者が想定外のエラーに合わないようにするため

####クラスプロパティ
クラスプロパティの定義の仕方を見て行きましょう。  

```objective-c
class ViewController: UIViewController {
	
	var variableNumber:Int?
	let unVariableNumber:Int = 3

	override func viewDidLoad() {
		super.viewDidLoad()

		var variableString:String		// String型の変数を定義
		let invariableString:String		// String型の定数を定義
	}
	<省略>
}
```

【ポイント】

* クラスプロパティ定義時は初期値が必要
* varは初期値がnilなので具体的な値を書く必要はない
* letは初期値にnilが入らないので、具体的な値を設定する必要がある

####クラスメソッド
クラスメソッドの定義の仕方を見て行きましょう。  

```objective-c
class ViewController: UIViewController {
	
	var variableNumber:Int?
	let unVariableNumber:Int = 3

	override func viewDidLoad() {
		super.viewDidLoad()

		println(setTestString("abc", b:"def"))
	}

	<省略>

	func setTestString(a:String, b:String) -> String {
		return a+b
	}
}
```

【ポイント】

* func メソッド名(第一引数名:第一引数の型, 第二引数名:第二引数の型, ...) -> 戻り値の型 { .... }
* 呼び出し側では、メソッド名(第一引数に渡す値, 第二引数名:第二引数に渡す値)

まだまだSwiftについて調べていきますが、記事が長くなってきたので本日はここまで。  

参考サイト:  

* [SwiftのOptional型を極める](http://qiita.com/koher/items/c6f446bad54442a28bf4)
* [オプショナル型。 〜 なんとなく付ける ! ? 撲滅 - SlideShare](http://www.slideshare.net/hasegawatomoki/ss-40424010)
* [[Swift] Optional 型についてのまとめ Ver2](http://qiita.com/cotrpepe/items/518c4476ca957a42f5f1)
* [[Swift] Swiftのoptional valueの便利さ /「?」と「!」でより堅牢なコードへ](http://qiita.com/nori0620/items/afa94cc42f3233f54bdc)
* [【Swift】Optional型を安全にunwrapしよう](http://qiita.com/mokumoku/items/5fef496f4e97a053e53e)

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

