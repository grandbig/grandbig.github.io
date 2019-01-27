---
layout: post
title: "CocoaPods v1.6.0でハマる『linker command failed with exit code 1』エラーの対応"
date: 2019-01-26 00:44
comments: true
categories: ios
---

### はじめに
今回はドハマリして解決に時間がかかった `linker command failed with exit code 1` についての解決方法のメモです。  

前提は、  

* `Xcode10.1`  
* `CocoaPods` のバージョンが `v1.5.3` だったのを `v1.6.0.rc1` にアップデートした  

とします。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### 現象
Git上のConflictを解消しようと思っていたら、いつの間にかエラーが出るようになってしまっていました。  
筆者の場合は `RealmSwift` を利用していたのですが、急に `Realm` に対して `linker command failed with exit code 1` と表示されるようになっていました。  

### 解決方法
筆者の場合、2段階でハマりました。  

1. `CocoaPods` が `sudoあり/なし` 両方でインストールしてしまっていた  
2. `v1.6.0.rc1` にアップデートしてしまっていた  

#### CocoaPodsを重複してインストールしてしまった
恐らく試行錯誤している間に誤ってインストールしてしまったのだと思います。  
`CocoaPods` の何かでハマった時に、 `CocoaPods` のバージョンを旧バージョンに下げるなどの記事が見つかったりしますが、  
知らず知らずのうちに重複してインストールしていると正しくバージョン管理ができません。  

よって怪しいなと思ったら、 `CocoaPods` を全てアンインストールしましょう。  

```objective-c
$ gem uninstall cocoapods
$ sudo gem uninstall cocoapods
```

アンインストール後に `pod --version` を実行してバージョンが非表示であれば、全て削除できています。  

#### v1.6.0.rc1にアップデートしてしまった
どうも `v1.6.0.rc1` が悪さをしているようでしたので、下記手順を実行して解決しました。  

1. “pod deintegrate”
2. “sudo gem install cocoapods-clean”
3. “pod clean”
4. Open the project and delete the “Pods” folder that should be red
5. “pod setup”
6. “pod install”

[XCConfig files are not properly deintegrated from the user project](https://github.com/CocoaPods/CocoaPods/issues/8091)で同様に苦しんでいる方々がいたので、コメントに書かれた上記の解決策で解消できました。  

### まとめ
`CocoaPods` 系ってたまにドハマリして時間取られることがあるので、気をつけねば...  
ということで本日はここまで。   

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
