---
layout: post
title: "Kotlinを使ってみよう！(2) : OkHttp編"
date: 2016-01-23 16:47
comments: true
categories: android kotlin
---

####今回のKotlinのお題は？
さて、久々にKotlinについて勉強をしたいと思います。  
今回のお題は[Android StudioでOkHttp3.0.1を使おう！](http://grandbig.github.io/blog/2016/01/19/android-okhttp/)にしたいと思います。  
では、早速見ていきましょう！  

####KotlinでOkHttp3.0.1を使うソースを書いてみる
まずは、OkHttp3.0.1を使ったソースを書いてみます。  
OkHttpの導入はJavaと同じです。  

build.gradle(app)に下記を追加しましょう。  

```java
dependencies {
	...
	compile 'com.squareup.okhttp3:okhttp:3.0.1'
}
```

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

Javaで紹介したときと同様に、Open Weather Map APIを使ったGETリクエストを利用します。  
OkHttpでは通信処理をメインスレッドで実行しようとするとエラーが発生してしまうので、`AsyncTask`を利用します。  
`AsyncTask`を継承したクラスを書きます。  

```java
package com.example.takahiro.simpleapplication

import android.os.AsyncTask

open class MyAsyncTask : AsyncTask<Void, Void, String>() {
	
	override fun doInBackground(vararg params: Void): String? {
		return null
	}
}
```

ここでKotlinの書き方を説明しましょう。  

* `open`
    * 他のクラスからの継承を許可するアノテーション  
    * `MainActivity`で`doInBackground`メソッドをオーバーライドするために設定  
　
* `vararg`
    * 可変長にしたい引数の名前に設定  
    * `donInBackground`は引数が1つ以上設定可能な可変長なものを持つため利用  

続いて、`MainActivity`を書いていきます。  

```java
package com.example.takahiro.simpleapplication

import android.os.AsyncTask
...
import okhttp3.OkHttpClient
import okhttp3.Request
import okhttp3.Response

class MainActivity:AppCompatActivity() {
	
	internal var client = OkHttpClient()
	
	override fun onCreate(savedInstanceState: Bundle?) {
		super.onCreate(savedInstanceState)
		setContentView(R.layout.activity_main)
		val toolbar = findViewById(R.id.toolbar) as Toolbar
		setSupportActionBar(toolbar)

		<省略>
		
		// GET Weatherボタンを取得して、クリック処理を実装
		val btn = findViewById(R.id.button) as Button
		btn.setOnClickListener {
			object: MyAsyncTask() {
				override fun doInBackground(vararg params: Void): String {
					var res: String = ""
					try {
						res = run("http://api.openweathermap.org/data/2.5/weather?APPID=<自身のAPPID>&q=Tokyo")
						val resJson = JsonObject(res)
						val weathers = resJson.getJSONArray("weather")
						val weather = weathers.getJsonObject(0)
						val description = weather.getString("description")
						Log.i("MainActivity", description)
						res = description
					} catch(e: IOException) {
						e.printStackTrace()
					} catch(e: JSONException) {
						e.printStackTrace()
					}
				}
			}.execute()
		}
	}
	
	fun run(url: String): String {
		val request = Request.Builder().url(url).build()

		val response = client.newCall(request).execute()
		return response.body().string()
	}
}
```

ここでもKotlinの書き方を説明します。  

* `object`
    * 匿名サブクラスのインスタンスの返却式  
    * setOnClickListenerの引数にMyAsyncTaskを匿名クラスとして指定するために利用  
    * Kotlinではインスタンス生成に`new`は利用しない  

さて、いかがでしたでしょうか？  
KotlinはSwiftに近い書式で書けるため、筆者にとっては非常にわかりやすいです。  
ただ、Javaとの互換性が非常に高いため、やはりそれなりに覚えなくてはならないことがありますね。  

これからも少しずつではありますが、Kotlinを見ていきたいと思います。  
といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
