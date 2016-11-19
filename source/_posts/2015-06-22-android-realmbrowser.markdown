---
layout: post
title: "AndroidでRealmで保存した値を確認しよう！"
date: 2015-06-22 00:23
comments: true
categories: android java
---

####Android Realm Browserを使おう
さて、[前回の記事](http://grandbig.github.io/blog/2015/06/20/android-realm/)で実装したRealmですが、値を確かめたいことがあると思います。  
その方法について見て行きましょう。  

[Android Realm Browser](https://github.com/dmytrodanylyk/realm-browser)を利用することでアプリ上で直接、値を確認することが可能です。  

実装手順  

１. GradleScripts > build.gradle(Module: app) のdependenciesを修正
`compile 'com.github.dmytrodanylyk.realm-browser:library:0.0.2'`を`dependencies { ... }`に追加します。  
２. `RealmFilesActivity`を起動する処理を追加

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

```java
// MainActivity.java
public class MainActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		...
		if(result.size() > 0) {
			...
			// Realmデータ閲覧用のActivityを起動
			RealmBrowser.getInstance().addRealmModel(Engineer.class);
			RealmFilesActivity.start(this);
		} else {
			...
		}
	}
	...
}
```

さて、これでアプリを実行してみましょう。  
下記のような画面を見ることができるはずです。  

![Realmデータ確認画面](/images/android-realmbrowser.png)  

####ADBで確認する方法
他にも確認する方法があります。  
Android StudioをPC上で使えるように設定するにあたって、意識せずにADBというツールがインストールされています。  
このADBコマンドを利用することでデバイスの中身を直接見ることが可能となります。  

ADB設定手順  

１. `.bashrc`ファイルにADBツールのパスを設定
`.bashrc`ファイルに`export PATH=$PATH:/Users/<ユーザ名>/Library/Android/sdk/platform-tools`とパスを設定しましょう。  
２. `.bashrc`ファイルの変更を反映
`source ~/.bashrc`を実行

これでADBコマンドを利用可能になりました。  
続いて下記手順で、Realmデータの中身を見て行きましょう。  

```java
$ adb shell
$ run-as com.example.takahiro.realmsample
$ cd files/
$ cat default.realm
```

日本語を保存している場合は文字化けして見えますが、保存されていることは確認できます。  
本当は`adb pull`コマンドを使ってPCに`default.realm`を持ってきて、Realm Browserアプリで中身を見たいのですが、  
今のところうまくコピーできず見れていません...。  
(因みに端末はNexus6を利用しています。)  

とりあえず、`Android Realm Browser`で見るしかないかな〜といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
