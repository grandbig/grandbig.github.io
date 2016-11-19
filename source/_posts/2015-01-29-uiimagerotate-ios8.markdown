---
layout: post
title: "iOS8で端末の向きに合わせて画像を回転させる方法について"
date: 2015-01-29 00:24
comments: true
categories: ios ios8
---

####端末の向きに合わせて画像を回転させよう！
iOS7.xまでは**willRotateToInterfaceOrientation**や**didRotateFromInterfaceOrientation**を用いて端末の向きが変わるときに、UIImageViewを回転させるといったことをやっていました。  
しかし、iOS8ではこれら2つのメソッドがdeprecatedになってしまいました。  
では、iOS8ではどうやって端末の向きに合わせてUIImageViewを回転させれば良いのでしょうか？  

ここではAutoLayoutを利用した方法を説明します。  

<!-- more -->

まず、前提として**『アプリの中央に配置した正方形の画像を端末の向きに合わせて回転させる』**とします。  
では早速手順を説明します。  

1: StoryboardにUIImageViewを配置しましょう。  
![UIImageViewを配置](/images/rotate_image_ios8_1.png)  

2: UIImageViewに画像をセットしましょう。  
![UIImageViewに画像をセット](/images/rotate_image_ios8_2.png)  

3: AutoLayoutを設定しましょう。  
設定するAutoLayoutは、下記5点です。  

* Aspect Ratio
* Center X Alignment
* Center Y Alignment
* Horizontal Space
* Vertical Space

Horizontal Space, Vertical Space, Aspect Ratioの設定は下記のようにできます。  
![AutoLayoutの設定](/images/rotate_image_ios8_3.png)  

ここで、Horizontal Space, Vertical Spaceは**Equal**ではなく**Greater**設定をします。  
設定は下図のようにHorizontal SpaceまたはVertical Spaceを選択して、右メニューのRelationを変更します。  
![AutoLayoutの設定](/images/rotate_image_ios8_5.png)  

Center X Alignment, Center Y Alignmentの設定は下記のようにできます。  
![AutoLayoutの設定](/images/rotate_image_ios8_4.png)  

全ての設定が終われば、下図のようになるでしょう。  
![AutoLayoutの設定](/images/rotate_image_ios8_6.png)  

実際にアプリを起動して端末を回転させてみると、  
端末がPortraitの場合は、  
![端末がPortraitの場合](/images/rotate_image_ios8_7.png)  

端末がLandscapeの場合は、  
![端末がLandscapeの場合](/images/rotate_image_ios8_8.png)  
となりました。  

う〜ん、だいぶ楽になりましたね。  
といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>



