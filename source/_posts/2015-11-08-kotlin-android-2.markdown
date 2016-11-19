---
layout: post
title: "Kotlinを使ってみよう！(1)"
date: 2015-11-08 23:54
comments: true
categories: android kotlin
---

###Kotlinを勉強しよう
さて、本日はKotlinを勉強してみたいと思います。  
勉強の題材としては、[Google Sign-In SDK for Androidを使ってみよう！](http://grandbig.github.io/blog/2015/11/08/google-sign-in-android/)で取り上げた **Google Sign-In SDKによるOAuth2.0認証** にしたいと思います。  
その中で筆者が抑えておくべきと思ったところを基礎からメモっていきます。  

(KotlinをAndroid Studioに導入する方法は以前書いた[Android StudioにKotlinを導入しよう！](http://grandbig.github.io/blog/2015/10/25/kotlin-android/)を参照ください。)  

では、早速、見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####Kotlinで変数/定数の書き方
最も基本的なことですが、変数/定数の書き方から異なります。  

Javaの場合  

```java
private final String TAG = "MainActivity";	// 定数
private GoogleApiClient mGoogleApiClient;	// 変数
```

Kotlinの場合  

```java
private val TAG:Int = "MainActivity"					// 定数
private var mGoogleApiClient:GoogleApiClient? = null	// 変数
```

ポイントは下記です。  

* 定数は`val`で変数は`var`を使う  
* 定数は、val 定数名:型名 = 値  
* 変数は、var 変数名:型名 = 値  
* 初期値未定の変数は型名の後ろに`?`をつけて、`null`を代入  
これをしないと『Property must be initialized or be abstract』エラーが発生する

####Kotlinでのstatic定数の書き方
Kotlinでは`static`は使いません。  

Javaの場合  

```java
private static final String TAG = "MainActivity";
private static final int RC_SIGN_IN = 9001;
```

Kotlinの場合  

```java
companion object {
	val TAG = "MainActivity"
	val RC_SIGN_IN = 9001
}
```

####Kotlinでimplementの書き方
Google Sign-In SDKでOAuth2.0認証をする際に、`GoogleApiClient.OnConnectionFailedListener`と`View.OnClickListener`のインターフェースを継承していました。  

Javaの場合  

```java
public class MainActivity extends AppCompatActivity implements GoogleApiClient.OnConnectionFailedListener, View.OnClickListener {
	...
}
```

Kotlinの場合  

```java
class MainActivity : AppCompatActivity(), GoogleApiClient.OnConnectionFailedListener, View.OnClickListener {
	...
}
```

Kotlionでは`implements`ワードは書かずに`extends`したクラスの後ろに`,`区切りで書きます。  

####Kotlinでキャストの書き方
Google Sign-Inを実行するために置いていたボタンをソース上で取り出すときに`SignInButton`でキャストしていました。  

Javaの場合  

```java
SignInButton signInButton = (SignInButton) findViewById(R.id.button_sign_in);
```

Kotlinの場合  

```java
var signInButton:SignInButton = findViewById(R.id.button_sign_in) as SignInButton
```

Kotlinでは`as`を使ってキャストします。  

####KotlinでOverrideの書き方

Javaの場合  

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
	...
}
```

Kotlinの場合  

```java
override fun onCreate(savedInstanceState: Bundle?) {
	...
}
```

Kotlinでは、`@`は不要で`fun`の前に書きます。  
(Swiftと違って、`func`とは書かないんです。)  

####Kotlinでメソッドの戻り値の書き方

Javaの場合  

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
	...
}
```

Kotlinの場合  

```java
override fun onCreateOptionsMenu(menu: Menu): Boolean {
	...
}
```

『fun メソッド名(引数名: 引数の型): 戻り値の型』 と書きます。  

####Kotlinでswitch文の書き方

```java
@Override
public void onClick(View v) {
	switch (v.getId()) {
		case R.id.button_sign_in:
			signIn();
			break;
	}
}
```

Kotlinの場合  

```java
override fun onClick(view:View) {
	when(view.id) {
		R.id.button_sign_in -> {
			signIn()
		}
	}
}
```

Kotlinでは`switch`ワードを使いません。`when`を使います。  
`case 条件:`という書き方が`条件 ->`になります。  

さて、今回気になったのはこんなところです。  
これからもどんどんKotlinでの書き方に慣れていきたいと思います。  

因みに、今回のソースは、[GitHub](https://github.com/grandbig/KotlinGoogleSignInSample)に上げてあります。  
ご自由にお使いください。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
