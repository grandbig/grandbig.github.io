---
layout: post
title: "library not found for -lPods"
date: 2016-06-11 22:26
comments: true
categories: ios
---

####library not found for -lPods
最近このエラーにハマりました。  
CocoaPodsで外部ライブラリをインストールして、いざXcodeプロジェクトを開いてビルドをしてみると...  

```objective-c
linker command failed with exit code 1
(use -v to see invocation)
ld: library not found for -lPods
```

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

なんてエラーが発生して失敗してしまいました。  
はっきり言って、このエラー内容を見ても、  
『CocoaPodsで取り込んだはずのライブラリが見つけられていない』  
ということしかわかりません。  

結局どう解決したかというと...  
プロジェクト内に含まれる`libPods.a`と`libPods-XXX.a` (XXX: Target名)を全て削除しました。  
削除しても`pod install`で問題なくこれらファイルは復活するのでご安心を。  

どうも、これらファイルが残った状態だと`pod install`しても新たにファイルが上書きされるわけではないようなんです。  

チーム開発するときはGitに古い`libPods.a`, `libPods-XXX.a`が上がっていないか十分に注意しましょう。  
心配な場合は`.gitignore`に`*.xcworkspace`と共に`libPods.a`, `libPods-XXX.a`を記載しておくのも手かもしれません。  

と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
