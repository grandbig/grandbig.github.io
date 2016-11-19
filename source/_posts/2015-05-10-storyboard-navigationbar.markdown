---
layout: post
title: "StoryboardでNavigationBarを表示する"
date: 2015-05-10 22:04
comments: true
categories: ios
---

####StoryboardでNavigationBarを表示する方法
さて、今日も忘れやすい内容なのでメモメモ...。  
最近はXIBファイルよりもStoryboardを利用するシーンが多くなっているかと思います。  
そこで、本日はStoryboardでNavigationBarを表示する方法について説明したいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

方法は実に簡単です。  
Main.storyboardを開いて、 **メニューのEditor > Embed In > Navigation Controller** を選択してください。
すると、これまでViewControllerが1つだけだった状態からNavigation Controllerが追加されます。  
また、初めからあったViewControllerにもNavigationBarが表示されたことがわかると思います。  
これは正確には **Navigation Item** が追加された状態です。このNavigation Itemを編集すれば、任意のタイトルを付与することができますし、Backボタンの名称も決められます。さらに、 **prompt** (説明文)を表示することもできます。  

Backボタンではなく、好きなアクションを起こさせるためのボタンを追加する場合には、右メニューバーからButtonを追加すればOKです。自動的にBar Button Itemが追加されます。  
すると、下図のような構成になるはずです。  

![ViewControllerの構成](/images/storyboard-navigationbar.png)  

Storyboardの画面的には下図になります。  

![Storyboard](/images/storyboard-navigationbar2.png)  

といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

