---
layout: post
title: "CocoaPods v1.0.0のメモ"
date: 2016-06-04 22:26
comments: true
categories: ios
---

####CocoaPodsをv1.0.0にしたらエラーが出てしまった！
さて、本日はメモ書きです。  
先日、筆者はCocoaPodsのバージョンを **1.0.0** にアップデートしました。  
で、早速、`pod install`を実行してみたところ、下記のようなエラーが...  

```objective-c
[i] he dependency `AFNetworking (~> 2.6.3)` is not used in any concrete target.
The dependency `CocoaLumberjack (~> 1.9.2)` is not used in any concrete target.
The dependency `MagicalRecord (~> 2.3.2)` is not used in any concrete target.
...
```

<!-- more -->

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

おっと、何だ何だ？？となったわけですが、ググってみたらすぐにわかりました。  
どうやら、Targetにインストール対象を定義することがMUSTになったようなのです。  

よって、  

```objective-c
pod 'AFNetworking', '~>2.6.3'
pod 'CocoaLumberjack', '~> 1.9.2'
pod 'MagicalRecord', '~>2.3.2'
```

と書いていたものを  

```objective-c
target 'MyProject' do
	pod 'AFNetworking', '~>2.6.3'
	pod 'CocoaLumberjack', '~> 1.9.2'
	pod 'MagicalRecord', '~>2.3.2'
end

target 'MyProjectTests' do
	pod 'AFNetworking', '~>2.6.3'
	pod 'CocoaLumberjack', '~> 1.9.2'
	pod 'MagicalRecord', '~>2.3.2'
end

target 'MyProjectUITests' do
	pod 'AFNetworking', '~>2.6.3'
	pod 'CocoaLumberjack', '~> 1.9.2'
	pod 'MagicalRecord', '~>2.3.2'
end
```

のように具体的に指定することが必要となります。  
しかし、共通しているインストール対象を何度も書くのは無駄なので...  

```objective-c
def common_pods
	pod 'AFNetworking', '~>2.6.3'
	pod 'CocoaLumberjack', '~> 1.9.2'
	pod 'MagicalRecord', '~>2.3.2'
end

target 'MyProject' do
	common_pods
end

target 'MyProjectTests' do
	common_pods
end

target 'MyProjectUITests' do
	common_pods
end
```

と簡略化することができます。便利ですね！  
ということで今更かつ簡単なメモ書きでした。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

