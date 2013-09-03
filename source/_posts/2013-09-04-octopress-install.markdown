---
layout: post
title: "Octopressインストールしてみました"
date: 2013-09-04 00:23
comments: true
categories: 
---

##Octopressとは
GitHub Pagesと組み合わせて、無料で簡単にブログが作れると噂のフレームワークです。
その噂の真相を確かめるべく、いざOctopressをインストールしてみました。

それがまさかこんな苦労するなんて夢にも思わず…

###Rubyのインストール
それではまず以下のコマンドを打ってみよう！
```
ruby --version
```

「なんだRubyなんて初めから入っているじゃないか」と油断してはいけません。
確かにMacにはデフォルトでRubyがインストールされています。
しかし、Octopressを使うには1.9.3以上が必要になります。
恐らくデフォルトで入っているのは1.8.xだったのではないでしょうか？

ネットで調べたところ、1.9.3を利用している人が多かったので、ひとまず1.9.3のインストールを目指すことにしました。

####homebrewとFomulaを最新版にアップデート
```
brew update
```
ここでopensslやreadlineなど必要なものをbrewでインストールと書いてあるサイトがあったのですが、
rubyインストール時に必要なものをインストールしてくれるのでrvmのインストールに進みます。  
※筆者は『brew install openssl』を実行してopensslのみ別でインストールしてしまったため後々path周りでハマってしまいました…。

####rvmをインストール
```
\curl -L https://get.rvm.io | bash -s stable
source ~/.bashrc
```

rvm -vでバージョンを確認できればrvmが無事インストールされたことになります。
```
rvm -v
rvm 1.22.3 (master) by Wayne E. Seguin <wayneeseguin@gmail.com>, Michal Papis <mpapis@gmail.com> [https://rvm.io/]
```

