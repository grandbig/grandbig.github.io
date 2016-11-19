---
layout: post
title: "tabBarControllerとUINavigationControllerを同時に使いたい！"
date: 2016-01-17 22:18
comments: true
categories: ios
---

####tabBarControllerとUINavigationControllerを両立させよう！
本日はtabBarControllerとUINavigationControllerを両方使いたい場合の方法を説明します。  
方法は簡単です。  

**tabBarControllerを先に追加して、**  
**その子要素にUINavigationControllerを追加してください。**  

具体的に図で説明します。  

１．storyboardでtabBarControllerを追加  
XcodeメニューのEditor > Embed in > Tab Bar Controller を選択してください。  
![tabBarControllerの追加](/images/tabBarController_1.png)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

すると、下図のようにstoryboardにTab Bar Controllerが追加されます。  
![storyboardに表示されます](/images/tabBarController_2.png)  

このままでは、tabBarにitem(項目)が１つしかないので、ViewControllerを追加しましょう。  

２．ドラッグ＆ドロップでstoryboardにUIViewControllerを追加  
右メニューから **View Controller** をドラッグ＆ドロップしましょう。  
![View Controllerをドラッグ＆ドロップ](/images/tabBarController_3.png)  

３．tabBarControllerと追加したUIViewControllerを紐付ける  
Tab Bar Controller上でcontrolキーを押しながら、追加したView Controllerの方へ線を伸ばして紐付けます。  
![Tab Bar ControllerとView Controllerのひも付け](/images/tabBarController_4.png)  

これにより、Tab Bar Controllerに1つ項目が増えて、View Controllerまでひも付けが伸びていることを確認できます。  
![項目の追加完了](/images/tabBarController_5.png)  

続いて、UINavigationControllerを追加していきましょう。  

４．子要素のView ControllerにUINavigationControllerを追加  
UINavigationControllerを追加したいView Controllerを選択して、  
XcodeメニューのEditor > Embed in > Navigation Controller を選択してください。  
![UINavigationControllerの追加](/images/tabBarController_6.png)  

結果、Tab Bar ControllerとView Controllerの間にNavigation Controllerが追加されたことがわかると思います。  
![Navigation Controllerが追加されました](/images/tabBarController_7.png)  

別の子要素のView ControllerにもNavigation Controllerが必要な場合は、  
そのView Controllerに対して同じ操作を繰り返し実施してください。  

storyboard上ではUINavigationControllerを先に追加して、  
tabBarControllerを追加することもできます。  
ですが、UINavigationControllerの戻るボタンの非表示やカスタム化などをしようとすると、  
どうもうまくいきません...。  
なので、一見無駄に見えるかもしれませんが、この方法を試してみることをオススメします。  

本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
