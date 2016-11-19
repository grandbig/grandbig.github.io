---
layout: post
title: "Octopressインストールしてみました"
date: 2013-09-04 00:23
comments: true
categories: octopress
---

##Octopressとは
GitHub Pagesと組み合わせて、無料で簡単にブログが作れると噂のフレームワークです。
その噂の真相を確かめるべく、いざOctopressをインストールしてみました。

それがまさかこんな苦労するなんて夢にも思わず…

<!--more-->

###Rubyのインストール
それではまず以下のコマンドを打ってみよう！
```
$ ruby --version
```

「なんだRubyなんて初めから入っているじゃないか」と油断してはいけません。
確かにMacにはデフォルトでRubyがインストールされています。
しかし、Octopressを使うには1.9.3以上が必要になります。
恐らくデフォルトで入っているのは1.8.7だったのではないでしょうか？

ネットで調べたところ、1.9.3を利用している人が多かったので、ひとまず1.9.3のインストールを目指すことにしました。

####homebrewとFomulaを最新版にアップデート
```
$ brew update
```
ここでopensslやreadlineなど必要なものをbrewでインストールと書いてあるサイトがあったのですが、
rubyインストール時に必要なものをインストールしてくれるのでrvmのインストールに進みます。  
※筆者は『brew install openssl』を実行してopensslのみ別でインストールしてしまったため後々path周りでハマってしまいました…。(詳細は後ほど)

####rvmをインストール
```
$ \curl -L https://get.rvm.io | bash -s stable
$ source ~/.bashrc
```

rvm -vでバージョンを確認できればrvmが無事インストールされたことになります。
```
$ rvm -v
rvm 1.22.3 (master) by Wayne E. Seguin <wayneeseguin@gmail.com>, Michal Papis <mpapis@gmail.com> [https://rvm.io/]
```

###Octopressをインストール
さあ、いよいよOctopressをインストールしましょう。
```
$ git clone git://github.com/imathis/octopress.git octopress
```
問題なければ次の4行も余裕でパスできるはず！
```
$ cd octopress
$ gem install bundler
$ bundle install
rake install
```

筆者はgem install bundlerがうまくいきませんでした。  
エラーメッセージを見る限り、opensslが見つからないらしい…。なぜだ!?ちゃんとopensslをインストールしたはずなのに！  
ちゃんとバージョンだって確認できる!!  
```
$ openssl version
OpenSSL 0.9.8x 10 May 2012
```
もしやインストールの仕方が違ったからか!?
たぶん見るpathを変えれば良いんだろうけど…  
はい！アンインストール
```
$ brew uninstall openssl
```
そして、他のライブラリと同様にrvmでインストール
```
$ rvm pkg install openssl
```

これで見事4行をパスしました。

###GitHubの設定
さて、あともうひと踏ん張りです。  
次はGitHubの設定をしましょう。今回は既にGitHubアカウントを作成していたため、アカウント作成周りは省略します。(恐らく思いつきで作ったのだろう…)  
アカウント作成が終わったら、ホーム画面に移動するはずなので、右下から*New repository*を選択してGitHub page用のレポジトリを作りましょう！  
下の画像のようにRepository nameに*username.github.io*を入力だ！(usernameにはGitHubアカウント作成時に作ったユーザ名を入れて下さい。)  
![GitHubページでNew Repositoryを作成](/images/github_new_repository.png)  

###GitHub Pagesを設定
GitHubの設定で作成したレポジトリをGitHub Pagesに設定します。
```
$ rake setup_github_pages
Enter the read/write url for your repository
	(For example, 'git@github.com:your_username/your_username.github.io)
	or 'https://github.com/your_username/your_username.github.io')
Repository url:
```
※末尾に*.git*はつけないように注意

###静的ページを作成して、デプロイ
最後に下記2つのコマンドを叩けば、晴れてBlogのできあがり！！
```
$ rake generate
$ rake deploy
```

いかがでしたでしょうか？  
筆者もうまくいかずに何度か諦めそうになりましたが、なんとかBlogを書く領域まで辿りつけました。  
情弱プログラマーは日々精進が大切ですね。
ではまた次回！
