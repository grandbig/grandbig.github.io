---
layout: post
title: "Android StudioでCROSSWALKを使おう！"
date: 2015-03-28 13:57
comments: true
categories: android crosswalk webview java
---

####Androidでハイブリッドアプリを作ろう
以前、iOSではiOS8からWKWebViewという新しいWebViewが追加されたことを説明しました。  
では、Androidはどうなのでしょうか？  

Androidは4.3までWebKitベースのWebViewをアプリに組み込んでいました。  
しかし、Android4.4より、ChromiumベースのWebViewとなりました。ここで、大きく異なるのはレンダリングエンジンが **Blink** になったというところです。  
また、Android4.4まではChromiumがバージョンアップされたとしても、各メーカーがAndroidのOSバージョンアップを提供しなければ、最新のWebViewを利用することができませんでした。  

この現状を見たのかGoogleはAndroid5.0以降、WebViewをOSから切り離し、Google Play経由でバージョンアップ可能としました。  
よって、メーカーがOSのバージョンアップをする/しないに関わらず、Google側の意志で最新のWebViewをユーザに提供可能となったのです。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

近年はAndroid2.xを使っているユーザもだいぶ減少してきました。  
が、未だ、Android4.0〜4.3の端末を利用しているユーザは相当数、存在しています。  
では、4.x〜5.0.xまでの端末全てに対応するハイブリットアプリを作成するためには、各バージョンに気をつけつつ、開発するしかないのでしょうか？  

本記事では、 **CROSSWALK** に注目し、上記の問題に対応することとしました。  

####CROSSWALKを組み込もう
まずは、CROSSWALKのレポジトリを追加できるようにしましょう。  

```java
// build.gradle(Module: app)
apply plugin: 'com.android.application'

android {
	....
}

// ここを追加
repositories {
	maven {
		url 'https://download.01.org/crosswalk/releases/crosswalk/android/maven2'
	}
}

dependencies {
	compile fileTree(dir: 'libs', include: ['*.jar'])
	compile 'com.android.support:appcompat-v7:21.0.3'
	compile 'org.xwalk:xwalk_core_library:10.39.235.15'		// ここも追加
}

```

次に、CROSSWALKのWebViewである **XWalkView** を画面に組み込みましょう。  

```xml
// activity_main.xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
	android:layout_height="match_parent" android:paddingLeft="0dp"
	android:paddingRight="0dp"
	android:paddingTop="0dp"
	android:paddingBottom="0dp" tools:context=".MainActivity">

	<!-- XWalkViewを追加 -->
	<org.xwalk.core.XWalkView
		android:id="@+id/xwalkWebView"
		android:orientation="vertical"
		android:layout_width="fill_parent"
		android:layout_height="fill_parent"
		android:background="#000000"
		/>

</RelativeLayout>
```

続いて、AndroidManifest.xmlにWebViewを使うための設定を書きましょう。  

```xml
// AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.takahiro.crosswalksample" >

	<!-- 追加 -->
	<uses-permission android:name="android.permission.INTERNET" />
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
	<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />

	....

</manifest>
```

そして、メイン画面にWebViewの設定を書き込みましょう。  

```java
// MainActivity.java

package com.example.takahiro.crosswalksample;

import android.support.v7.app.ActionBarActivity;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import org.xwalk.core.XWalkView;

public class MainActivity extends ActionBarActivity {
	
	private XWalkView mXWalkView;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		mXWalkView = (XWalkView) findViewById(R.id.activity_main);
		mXWalkView.load("https://www.google.co.jp", null);
	}

	@Override
	protected void onPause() {
		super.onPause();
		if (mXWalkWebView != null) {
			mXWalkWebView.pauseTimers();
			mXWalkWebView.onHide();
		}
	}
	
	@Override
	protected void onResume() {
		super.onResume();
		if (mXWalkWebView != null) {
			mXWalkWebView.resumeTimers();
			mXWalkWebView.onShow();
		}
	}

	@Override
	protected void onDestroy() {
		super.onDestroy();
		if (mXWalkWebView != null) {
			mXWalkWebView.onDestroy();
		}
	}
}

```

これでAndroid4.x 〜 5.xで同じChromiumベースのWebViewを利用することができるはずです。  
といったところで本日はここまで。  

参考:  

* [Embedding Crosswalk in Android Studio](https://diego.org/2015/01/07/embedding-crosswalk-in-android-studio/)
* [Embedding the Crosswalk Project](https://crosswalk-project.org/documentation/embedding_crosswalk.html)
* [XWalkView Reference](https://crosswalk-project.org/apis/embeddingapidocs_v3/org/xwalk/core/XWalkView.html#addJavascriptInterface(java.lang.Object, java.lang.String)

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
