---
layout: post
title: "Android Studioでvolley frameworkを利用するまで"
date: 2015-05-24 20:50
comments: true
categories: android java
---

####volley frameworkを使ったプロジェクトをGitHubにアップする
本日はAndroidについて書きます。  
Androidアプリを開発する上で、通信系のframeworkを使うなら、[volley framework](https://android.googlesource.com/platform/frameworks/volley)を選択すると思います。  
非常に便利なframworkである反面、導入が面倒だったりします。  
筆者はiOSエンジニアなので、[CocoaPods](https://cocoapods.org/)をよく使うのですが、如何に便利なツールなのかがよくわかりました笑  

#####プロジェクトの作成
まずは、Android Studioでプロジェクトを作成して、GitHubにアップしましょう。  

1. Android Studioでプロジェクトを作成(VolleySampleを作成)
2. GitHubで新規repogitoryを作成
3. Android Studioのプロジェクトのルート(VolleySample/)配下に移動して、下記コマンドを実行しましょう。

```java
$ echo "# VolleySample" >> README.md
$ git init
$ git add .
$ git commit -m "first commit"
$ git remote add origin https://github.com/grandbig/VolleySample.git
$ git push -u origin master
```

下記のようになればOKです。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

![プロジェクトをGitHubにアップ](/images/android-volley-1.png)  

#####volleyの導入
次にvolley frameworkを自身のプロジェクトに導入します。  
導入にはgit submoduleを使います。  

先ほど作成したAndroid Studioプロジェクトのルート(VolleySample/)配下で下記コマンドを実行  

```java
$ git submodule add https://android.googlesource.com/platform/frameworks/volley modules/volley
```

これにより、VolleySample/.gitmodulesが作成されます。  
中身はというと、  

```java
[submodule "modules/volley"]
	path = modules/volley
	url = https://android.googlesource.com/platform/frameworks/volley
```

になっているはずです。
続いて、`settings.gradle(Project Settings)`に下記1行を追加します。  

```java
include ':modules:volley'
```

![settings.gradleの設定](/images/android-volley-2.png)  

そして、`build.gradle(Module: app)`の`dependencies`に1行追加しましょう。  

```java
dependencies {
	compile fileTree(dir: 'libs', include: ['*.jar'])
	compile 'com.android.support:appcompat-v7:21.0.3'
	compile project(':modules:volley')	.... ここを追加しましょう
}
```

![build.gradleの設定](/images/android-volley-3.png)  

ここまできたら、一旦、GradleをSyncさせましょう。  
足りないものがあればインストールを求められるはずなので、その場合はインストールしておきましょう。  

では、GitHubにこれまでの作業分を更新させましょう。  

![volleyを追加した状態でGitHubにアップ](/images/android-volley-4.png)  

#####volleyの利用
実際にvolleyを使ってみましょう。  
[Swift Alamofireライブラリを使ってみよう！](http://grandbig.github.io/blog/2015/05/24/swift-alamofire1/)と[Swift AFNetworkingライブラリを使ってみよう！](http://grandbig.github.io/blog/2015/05/24/swift-afnetworking/)で例として使ったOpen Weather Map APIをここでも利用します。  

```java
// MainActivity.java
package com.example.takahiro.volleysample;

import android.support.v7.app.ActionBarActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.Menu;
import android.view.MenuItem;

import com.android.volley.Request;
import com.android.volley.RequestQueue;
import com.android.volley.VolleyError;
import com.android.volley.Response;
import com.android.volley.toolbox.JsonObjectRequest;
import com.android.volley.toolbox.Volley;

import org.json.JSONObject;

public class MainActivity extends ActionBarActivity {
	
	private RequestQueue mRequestQueue;
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		if (mRequestQueue == null) {
			mRequestQueue = Volley.newRequestQueue(getApplicationContext());
		}

		// URLの指定
		String url = "http://api.openweathermap.org/data/2.5/weather?q=Tokyo,jp";

		mRequestQueue.add(new JsonObjectRequest(Request.Method.GET, url, null, new Response.Listener<JSONObject>() {
			@Override
			public void onResponse(JSONObject response) {
				// 通信に成功した場合
				Log.d("VolleySample", response.toString());
			}
		}, new Response.ErrorListener() {
			@Override
			public void onErrorResponse(VolleyError error) {
				// エラーが発生した場合
				Log.d("VolleySample", error.toString());
			}
		});
	}

	<省略>

}
```

因みに、`AndroidManifest.xml`にインターネットの通信許可を与えることを忘れずに。  

```java
// AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.takahiro.volleysample" >
	
	<uses-permission android:name="android.permission.INTERNET" />

	<application
		<省略>
	</application>
</manifest>
```

無事、通信が成功しましたね！  
ってところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
