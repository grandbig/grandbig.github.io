---
layout: post
title: "OS X Serverを使ったらMacがシステム終了しなくなった"
date: 2015-03-15 23:32
comments: true
categories: mac osx
---

####OS X ServerでXcodeの設定をしたら...
さて、本日は、急遽、Macbook Proがシステム終了しなくなった件についてのメモです。  
先日、OS X ServerのXcode周りの設定をいじっていました。  
すると、下記の現象が発生するようになってしまいました。  

* システム終了しようとすると、終了していないユーザがいるため、ユーザの切替を要求される
* システム終了しようとしても、Macbook Proがうんともすんとも言わない

<!-- more -->

これには困りました...。  
が、下記のコマンドを実行してOS X ServerでのXcodeの設定をリセットしたところ、これまで通り、システム終了できるようになりました。  

`sudo xcrun xcscontrol --reset`

OS X ServerでXcodeを設定する必要がないのであれば、上記コマンドでリセットしてしまって問題ないでしょう。  
あ〜良かった。  

参考:  

* [_xcsbuildd user preventing logout after running OS X Server](http://blog.stuart.shelton.me/archives/2563)
* [Apple Developper Forums](https://devforums.apple.com/message/1053273#1053273)

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

