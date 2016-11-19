---
layout: post
title: "Android Studio2.0: TabLayoutを使ってTabを画面最下部に配置しよう！"
date: 2016-02-11 14:08
comments: true
categories: android java
---

####Tabを最下部に配置する方法について
さて、本日は簡単な話なのですが、案外ハマったので、メモ代わりに書いておきたいと思います。  
以前、[Android Studio2.0: TabLayoutを使ってみよう！](http://grandbig.github.io/blog/2016/01/30/android-tablayout/)でTabLayoutを使ったTabデザインの実装方法について紹介させて頂きました。  
そのときはTabをToolbarのすぐ下に配置していましたが、画面最下部に配置したいことも出てくることでしょう。  
その方法について書きたいと思います。  

TabLayoutの配置は`content_main.xml`に下記のように書きます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

```java
// content_main.xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:app="http://schemas.android.com/apk/res-auto"
	xmlns:tools="http://schemas.android.com/tools"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:paddingBottom="@dimen/activity_vertical_margin"
	android:paddingLeft="@dimen/activity_horizontal_margin"
	android:paddingRight="@dimen/activity_horizontal_margin"
	android:paddingTop="@dimen/activity_vertical_margin"
	app:layout_behavior="@string/appbar_scrolling_view_behavior"
	tools:context="com.example.takahiro.tablayoutsample2.MainActivity"
	tools:showIn="@layout/activity_main">

	<android.support.v4.view.ViewPager
		android:id="@+id/pager"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:layout_above="@+id/tabs"/>

	<android.support.design.widget.TabLayout
		android:id="@+id/tabs"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:background="@color/colorPrimary"
		app:tabSelectedTextColor="@color/colorAccent"
		app:tabTextColor="#ffffff"
		android:layout_alignParentBottom="true"
		android:layout_alignParentLeft="true"
		android:layout_alignParentStart="true"/>

</RelativeLayout>
```

ポイントは、  

* `android.support.v4.view.ViewPager`に`android:layout_above="@+id/tabs"`設定  
* `android.support.design.widget.TabLayout`に`android:layout_alignParentBottom`, `android:layout_alignParentLeft`, `android:layout_alignParentStart`の3つをtrueで設定  

です。  
これにより、下図のような配置ができているはずです。  

![Tabを最下部配置](/images/android-tablayout-2.png)  

なんてことはない単純な話なのですが、**Android Design Support Library** に惑わされてしまいました。    
もっとAndroidアプリの特にUI・UX周りの開発に精進したいと思います。  

と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
