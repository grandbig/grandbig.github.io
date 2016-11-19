---
layout: post
title: "nodebrewで複数のnodeをインストールしよう"
date: 2014-01-19 23:14
comments: true
categories: node.js
---

###nodebrewでnodeをインストール
最近はNode.jsがかなりメジャーになってきた気がします。しかし、Node.jsは未だにバージョンが1.0に届いていないので、開発途中に最新バージョンが出てくることも十分あり得ます。そのたびにアンインストールをするわけにもいかないので複数のバージョンのNode.jsをインストールできるようにしておくと便利です。  
nvmを使う方法もあるのですが、筆者は簡単に設定ができるnodebrewを活用しています。  

<!--more-->

####設定手順
早速、設定手順を説明します。  
1\. nodebrewをインストール
```
curl -L git.io/nodebrew | perl - setup
```
2\. \.bash\_profileにパスを追加
```
export PATH=$HOME/.nodebrew/current/bin:$PATH
```
3\. \.bash\_profileを再読み込み
```
source ~/.bash_profile
```
4\. nodeのインストール
```
nodebrew install-binary v0.10.24
```
ただし、v0.8.6以前はバイナリからインストールできないので、下記のようにインストールする必要があります。
```
nodebrew install v0.6.12
```
5\. インストールされたnodeのバージョンを確認
```
nodebrew ls
```
6\. 利用したいnodeを指定
```
nodebrew use v0.10.24
```

以上。

