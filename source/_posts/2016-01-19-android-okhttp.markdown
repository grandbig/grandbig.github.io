---
layout: post
title: "Android StudioでOkHttp3.0.1を使おう！"
date: 2016-01-19 00:35
comments: true
categories: android java
---

####Volleyではなく、OkHttpを使おう！
さて、本日は久しぶりにAndroidの話です。  
これまでAndroidの通信ライブラリとして活躍してきた **Volley** が敬遠されている傾向があります。  
と言うのも、Volleyが依存しているApache Http ClientがAPI Level22からdeprecatedになったためです。  

では、今後は何を使っていけば良いのでしょうか？  
もちろん開発者自身で通信処理を0から実装することも悪いことではないのですが、  
やはり、少しでも開発効率を良くするためにオープンソースを利用したいところですよね。  

そこで、昨今、皆が使っているものとして[OkHttp](http://square.github.io/okhttp/)というものがあります。  
これは **HTTP** だけでなく、 **HTTP/2** も対応しているため、きちんと最新を追えているという安心感がある点も良い印象です。  

また、[クックパッド開発者ブログ](http://techlife.cookpad.com/entry/2015/06/25/093507)でも **OkHttp** への移行を進める旨が書かれていますので、知っていて損はなさそうです。  

今回は、OKHttpを使ったサンプルを載せたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####OKHttpの導入
Android StudioにOKHttpを導入しましょう。  
導入は超簡単です。  

```java
dependencies {
	...
	compile 'com.squareup.okhttp3:okhttp:3.0.1'
}
```

はい。以上です。  

####OKHttpでGETリクエストを実行
さて、GETリクエストを実行してみましょう。  
今回は下記のようなサンプルアプリを作成しました。  
![サンプルアプリ](/images/android-okhttp.png)  

まずは、`AsyncTask`を継承したクラスを下記のように作ります。  

```java
package com.example.takahiro.simpleapplication;

import android.os.AsyncTask;

/**
 * Created by kato_takahiro on 2016/01/19.
 */
public class MyAsyncTask extends AsyncTask<Void, Void, String> {

	// コンストラクタ
	public MyAsyncTask() {
		super();
	}

	@Override
	protected String doInBackground(Void... params) {
		return null;
	}
}
```

さて、`MainActivity.java`内で実行してみます。  
このとき、 **GET Weather** ボタンをタップしたときに天気情報を取得するものとします。  

```java
public class MainActivity extends AppCompatActivity {
	OkHttpClient client = new OkHttpClient();

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
		setSupportActionBar(toolbar);

		<省略>

		// GET Weatherボタンを取得して、クリック処理を実装
		Button btn = (Button) findViewById(R.id.button);
		btn.setOnClickListener(new View.OnClickListener() {
			@Override
			public void onClick(View view) {
				new MyAsyncTask() {
					@Override
					protected String doInBackground(Void... params) {
						String res = null;
						try {
							String result = run("http://api.openweathermap.org/data/2.5/weather?APPID=<自身のAPPID>&q=Tokyo");
							JSONObject resJson = new JSONObject(result);
							JSONArray weathers = resJson.getJSONArray("weather");
							JSONObject weather = weathers.getJSONObject(0);
							String description = weather.getString("description");

							res = description;
						} catch(IOException e) {
							e.printStackTrace();
						} catch(JSONException e) {
							e.printStackTrace();
						}

						return res;
					}
				}.execute();
			}
		}
	}

	// OKHttpを使った通信処理
	public String run(String url) throws IOException {
		Request request = new Request.Builder()
				.url(url)
				.build();

		Response response = client.newCall(request).execute();
		return response.body().string();
	}
}
```

忘れずに`AndroidManifest.xml`で通信許可しておきましょう。  

```java
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.takahiro.simpleapplication">

	<uses-permission android:name="android.permission.INTERNET" ></uses-permission>
	
	<省略>

</manifest>
```

まだ、GETしか試していませんが、割りと簡単に実装できますね。  

####注意点
初め、実装するにあたって、躓いたことを書いておきます。  
何となしに`OKHttp`を実装し、デバッグしたところ、`android.os.NetworkOnMainThreadException`が発生しました。  

これは、通信処理をメインスレッドで実行しようとしたために発生しました。  
そのため、`AsyncTask`を使って別スレッドで通信処理を実行するように改修する必要がありました。  

さて、今回は筆者用のメモの意味合いが強いのですが、Androidアプリの開発に関する記事も今度どんどん増やしていきたいと思います。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
