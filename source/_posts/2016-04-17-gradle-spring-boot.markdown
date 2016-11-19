---
layout: post
title: "Spring BootでSystem.inにハマったメモ"
date: 2016-04-17 23:13
comments: true
categories: springboot gradle java
---

####Spring BootをGradleでビルドしてハマったメモ
今日は超絶ハマって苦しんだ超初歩についてメモ書きです。  
Spring Bootに関する情報はあまり多くはないのですが、  
さらに困ったことにほとんどの書籍やネット情報ではMavenでビルドしているのでGradleに関する情報が乏しくてハマりました...。  

Javaでキーボード入力のために、  
`Scanner scanner = new Scanner(System.in);`を記述することがあります。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

これをメインファイルに記述した状態で`gradle bootRun`コマンドを実行しても、  
しれっと処理が流れてしまいました...。  
「なぜだ〜〜〜」と格闘すること数時間!?  

解決方法がわかりました。下記を`build.gradle`に追加します。  

```java
// build.gradle
bootRun {
	standardInput = System.in
}
```

これは`bootRun`タスクを実行したときに、  
`System.in`をInput処理として判別するように定義しています。  
もう一度`gradle bootRun`を実行してみると処理が途中でStopしてユーザ入力を受け付けるようになりました。  

いや〜ハマった...。  
本日は完全なるメモ書きでした。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

