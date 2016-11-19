---
layout: post
title: "Android Studio2.0: TableLayoutを使ってみよう！"
date: 2016-02-11 15:15
comments: true
categories: android java
---

####TableLayoutとは
`TabLayout`に続いて、本日は、`TableLayout`を使った画面作成をしたいと思います。  
Androidでの`TableLayout`はHTMLで言うところの`<table>`タグに当たり、  
iOSで言うところの`UITableView`が使いどころとしては近いと思います。  

筆者が思うに`TableLayout`は決まったレイアウトの実装には向いていますが、  
複雑かつオリジナリティの高い画面を作成する際に使うには難しいのかなと感じています。  
何はともあれ、実装に入っていきましょう。  

本記事で利用しているAndroid Studioは **Android Studio 2.0 Beta 3** になります。  
(Android Studio 1.5 以上であれば基本的に同じだと思います。)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####TableLayoutを使ってみよう！
順を追って説明していきます。  
目標として、下図のような画面作成を目指したいと思います。  

![TableLayoutを使った画面](/images/android-tablelayout.png)  

これは1行目と4行目は1つのカラムで、2行目と3行目は2つのカラムで構成しています。  

1． Android Studioで`Blank Activity`を選択して、プロジェクトを新規作成  
下記のようなプロジェクト構成ができあがります。  

```java
app
 ┣━━ manifests
 ┃    ┗━━ AndroidManifest.xml
 ┣━━ java
 ┃    ┣━━ com.example.takahiro.tablelayoutapplication
 ┃    ┃      ┗━━ MainActivity
 ┃    ┣━━ com.example.takahiro.tablelayoutapplication(androidTest)
 ┃    ┃      ┗━━ ApplicationTest
 ┃    ┗━━ com.example.takahiro.tablelayoutapplication(test)
 ┃           ┗━━ ExampleUnitTest
 ┗━━ res
      ┣━━ drawable
      ┣━━ layout
      ┃     ┣━━ activity_main.xml
      ┃     ┗━━ content_main.xml
      ┣━━ menu
      ┣━━ mipmap
      ┗━━ values
```

2．`content_main.xml`に`TableLayout`を追加  
続いて、`TableLayout`をレイアウトファイルに実装していきましょう。  

```java
// content_main.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:app="http://schemas.android.com/apk/res-auto"
	xmlns:tools="http://schemas.android.com/tools"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:paddingBottom="0dp"
	android:paddingLeft="0dp"
	android:paddingRight="0dp"
	android:paddingTop="0dp"
	android:background="#efeff4"
	app:layout_behavior="@string/appbar_scrolling_view_behavior"
	tools:context="com.example.ec_dev.tablelayoutapplication.MainActivity"
	tools:showIn="@layout/activity_main">

	<!-- 構成要素が多くなる場合を踏まえて、画面スクロールを可能にしておく -->
	<ScrollView
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:id="@+id/scrollView">
		
		<TableLayout
			android:layout_width="match_parent"
			android:layout_height="wrap_content"
			android:stretchColumns="0">
			
			<!-- 1行目 -->
			<TableRow
				android:layout_width="match_parent"
				android:layout_height="match_parent"
				android:paddingBottom="8dp"
				android:paddingTop="8dp">
				
				<TextView
					android:layout_width="wrap_content"
					android:layout_height="wrap_content"
					android:textAppearance="?android:attr/textAppearanceLarge"
					android:text="@string/top_message"
					android:layout_span="2"
					android:id="@+id/textView"
					android:textSize="18sp"
					android:paddingRight="8dp"
					android:paddingLeft="8dp"/>
			</TableRow>
			
			<!-- 2行目 -->
			<TableRow
				android:layout_width="match_parent"
				android:layout_height="match_parent"
				android:background="#ffffff"
				android:paddingBottom="8dp"
				android:paddingTop="8dp">

				<TextView
					android:layout_width="wrap_content"
					android:layout_height="44dp"
					android:text="@string/androidText"
					android:id="@+id/textView2"
					android:textSize="20sp"
					android:gravity="center_vertical"
					android:paddingRight="8dp"
					android:paddingLeft="@dimen/8dp"/>

				<ImageButton
					android:layout_width="44dp"
					android:layout_height="44dp"
					android:id="@+id/imageButton"
					android:src="@drawable/ic_android_white_18dp"
					android:background="@color/colorAccent"
					android:layout_marginRight="8dp"/>
			</TableRow>
			
			<-- 3行目 -->
			<TableRow
				android:layout_width="match_parent"
				android:layout_height="match_parent"
				android:background="#ffffff"
				android:paddingBottom="8dp"
				android:paddingTop="8dp">

				<TextView
					android:layout_width="wrap_content"
					android:layout_height="44dp"
					android:text="@string/animalText"
					android:id="@+id/textView2"
					android:textSize="20sp"
					android:gravity="center_vertical"
					android:paddingRight="8dp"
					android:paddingLeft="8dp"/>

				<ImageButton
					android:layout_width="44dp"
					android:layout_height="44dp"
					android:id="@+id/imageButton"
					android:src="@drawable/ic_pets_white_18dp"
					android:background="@color/colorAccent"
					android:layout_marginRight="8dp"/>
			</TableRow>
			
			<!-- 4行目 -->
			<TableRow
				android:layout_width="match_parent"
				android:layout_height="match_parent"
				android:paddingBottom="8dp"
				android:paddingTop="8dp">
				
				<TextView
					android:layout_width="wrap_content"
					android:layout_height="wrap_content"
					android:textAppearance="?android:attr/textAppearanceLarge"
					android:text="@string/last_message"
					android:layout_span="2"
					android:id="@+id/textView"
					android:textSize="18sp"
					android:paddingRight="8dp"
					android:paddingLeft="8dp"/>
			</TableRow>
		</TableLayout>
	</ScrollView>
</LinearLayout>
```

ポイントは下記です。  

* `TableLayout`に`android:stretchColumns="0"`を設定して、0番目のカラム幅を極力大きくする  
* 1行目と4行目`TableRow`の`TextView`は`android:layout_span="2"`を設定して、カラム2つ分とする  

その他、必要なケースに合わせてデザインを整えましょう。  

因みに、`activity_main.xml`は以下にしています。  

```java
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:app="http://schemas.android.com/apk/res-auto"
	xmlns:tools="http://schemas.android.com/tools"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:fitsSystemWindows="true"
	tools:context="com.example.takahiro.tablelayoutapplication.MainActivity">

	<android.support.design.widget.AppBarLayout
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:theme="@style/AppTheme.AppBarOverlay">

		<android.support.v7.widget.Toolbar
			android:id="@+id/toolbar"
			android:layout_width="match_parent"
			android:layout_height="?attr/actionBarSize"
			android:background="?attr/colorPrimary"
			app:popupTheme="@style/AppTheme.PopupOverlay" />

	</android.support.design.widget.AppBarLayout>

	<include layout="@layout/content_main" />
</android.support.design.widget.CoordinatorLayout>
```

デフォルトでは`FloatingActionButton`が作成されますが、不要なので削除しています。  

合わせて、`strings.xml`も編集しています。  

```java
<resources>
	<string name="app_name">TableLayoutApplication</string>
	<string name="action_settings">Settings</string>
	<string name="top_message">Congratulation!!</string>
	<string name="androidText">Android</string>
	<string name="animalText">Animals</string>
	<string name="last_message">
		- this page is simple sample.\n- this app is simple sample application.\n- GO! GO!
	</string>
</resources>
```

レイアウトを作成している途中、レイアウトが想定していない形になったり、崩れてしまったりすることがあるかもしれません。  
そんなときはレイアウトファイル上でリフレッシュボタンをクリックして更新しましょう。  

![レイアウトのリフレッシュ](/images/android-tablelayout-2.png)  

さて、如何だったでしょうか？
今回の記事で紹介したProjectのソースは[GitHub grandbig/TableLayoutSample](https://github.com/grandbig/TableLayoutSample)に載せてあります。  
ご自由にご利用ください。  

因みに、今回はレイアウトの作成に注目して記事を書きました。  
そのため、ボタンアクションなど必要な場合は`MainActivity.java`に処理を追加してください。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
