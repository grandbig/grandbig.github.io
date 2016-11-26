---
layout: post
title: "Octopress Blogを別PCに移行しようとして苦労した話"
date: 2016-11-20 23:55
comments: true
categories: octopress
---

###新しいPCを購入したので、Octopress Blogを移行してみました
さて、本日は新しく購入したMacbook Proに本ブログの投稿環境を移行した話を書きます。  
何気に今まで目を背けていたことが仇となって結構苦労しました(汗)  

今後、PCを買い換えることがまたあると思うのでメモ代わりに残しておこうと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

１．ローカルとリモートのソースを合わせる  
当然ではあるのですが、Gitで管理しているローカルとリモートのソースを合わせます。  
筆者は実は2年間くらい放置していたので、結構、デグレっててたいへんで、しかもこれが尾を引くことに...  

２．`source` ブランチをクローンする  
最新にした`source`ブランチを新しいMacbook Pro側でクローンします。  

```
$ git clone -b source https://github.com/XXX/XXX.github.io.git
```

３．`master`ブランチを`_deploy`ディレクトリとしてクローンする  
ブランチを`master`に変更することを忘れずに！  

```
$ cd XXX.github.io
$ git clone https://github.com/XXX/XXX.github.io.git _deploy
```

４．`bundler`をインストールする  

```
$ gem install bundler
```

５．GitHub Pagesを設定する  

```
$ bundle install
$ rake setup_github_pages
Enter the read/write url for your repository
(For example, 'git@github.com:your_username/your_username.github.com):
```

自分のGitHub Pagesを入力しましょう。  

６．GitHub > Settings > SSH Keys に公開鍵を登録する  
まずは下記コマンドで秘密鍵、公開鍵を作成します。  

```
$ ssh-keygen
```

`~/.ssh`配下に`id_rsa.pub`と`id_rsa`が作成されているので、`id_rsa.pub`を開いて中身をコピーします。  
それを GitHub > Settings > SSH Keys で New SSH Key を選択して、登録します。  

これで、  

```
$ ssh -T git@github.com
Hi username! You've successfully authenticated...
```

という結果が得られます。  

７．強制プッシュを設定する  
`rake deploy`すると下記エラーが出てしまうようになり、かなり悩みました...  

```
## Pushing generated _deploy website
To https://github.com/XXX/XXX.github.io
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'https://github.com/XXX/XXX.github.io'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

Conflictが起こっているわけではないのに...  
悩んだ挙句、その場しのぎで強制プッシュを設定するようにしました。  
設定は`Rakefile`を変更することで可能です。  
これまで下記の設定がなされていたので、

```
system "git push origin #{deploy_branch}"
```

これを下記に変えます。  

```
system "git push origin +#{deploy_branch}"
```

これでプッシュができるようになり、`rake deploy`が成功したので、ブログも無事に更新できました。  
このままで良いとは思えないものの、他に解決方法わからず...一旦これで良いかな笑  

といったところで本日はここまで。  

参考ページ  

* [GitHub Help: Error: Permission denied (publickey)](https://help.github.com/articles/error-permission-denied-publickey/)  
* [rake gen_deploy rejected in Octopress](http://stackoverflow.com/questions/17609453/rake-gen-deploy-rejected-in-octopress)  
* [複数の環境で Octopress を使ってブログを書く](http://momota.github.io/blog/2016/01/29/octopress/)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
