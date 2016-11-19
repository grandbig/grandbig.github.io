---
layout: post
title: "Android StudioにKotlinを導入しよう！"
date: 2015-10-25 19:12
comments: true
categories: android kotlin
---

####AndroidアプリはKotlinで書く時代！？
さて、本日は久々にAndroidについて書きたいと思います。  
**Kotlin** はどの程度、世に普及しているのでしょうか？ほとんどのAndroidアプリ開発者はJavaで開発していると思うのですが、今後はiOSのObjective-C/Swiftのように開発言語としてJava/Kotlinを選択することになるかもしれません。  

KotlinはAndroid StudioのベースとなっているIntelliJを開発しているJetBrains社が開発したJVM言語であるため、Android Studioへの導入も簡単で親和性も高いらしいです。  
また、iOSアプリをSwiftで開発していた開発者は書き方が似ているKotlinの方が開発が幾分か楽かもしれません。  
コード補完や参照ジャンプなどの基本的なIDEサポートもあるのでAndroid Studioでも問題なくKotlinを使えるのではないでしょうか。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####Kotlinの導入方法
Kotlinの導入方法を書いていきます。と言ってもほとんどないです笑。  

【導入手順】  

1．Android Stuioを起動  
2．Android Studio > Preferences > Plugins > Install JetBrains plugin...を選択  
3．リストからKotlinを選択して、Install pluginを選択
![Kotlinのインストール](/images/kotlin_install_1.png)  

4．インストール完了後, Android Studioを再起動  
![Android Studioの再起動](/images/kotlin_install_2.png)  

5．javaパッケージをkotlinパッケージにリネーム  
javaパッケージを右クリック > Refactor > Rename を選択  
最終的に下図のようになればOKです。  
![ペッケージ名の変更](/images/kotlin_install_3.png)  

6．javaファイルをkotlinファイルに変換  
javaファイルを選択して、Code > Convert Java File to Kotlin File を選択  
MainActivity(メインファイル)だけでなくApplicationTestも変換しておきましょう。  
![kotlinファイルに変換](/images/kotlin_install_4.png)  

7．build.gradle(app)をkotlin用に修正
Tools > Kotlin > Configure Kotlin in Project を選択すると、build.gradle(app)ファイルにKotlin対応用の記述が自動で追加されます。  
筆者の場合は下記のような感じになりました。  

```java
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'

android {
	compileSdkVersion 21
	buildToolsVersion "22.0.1"

	defaultConfig {
		applicationId "com.example.kato_takahiro.kotlinsample"
		minSdkVersion 21
		targetSdkVersion 21
		versionCode 1
		versionName "1.0"
	}
	buildTypes {
		release {
			minifyEnabled false
			proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
		}
	}
	sourceSets {
		main.java.srcDirs += 'src/main/kotlin'
	}
}

dependencies {
	compile fileTree(dir: 'libs', include: ['*.jar'])
	testCompile 'junit:junit:4.12'
	compile 'com.android.support:appcompat-v7:21.0.3'
	compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
}
buildscript {
	ext.kotlin_version = '1.0.0-beta-1038'
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
	}
}
repositories {
	mavenCentral()
}
```

8．ビルドしてデバッグ  
ここまでくれば設定は完了したはずなので、Build > Clean Projectしてエラーが発生しないかチェック  
問題なければ、実機を繋いでデバッグしちゃいましょう！  

####Kotlinでコードを書いてみよう！
さて、使えることにしただけでは何なので、以前書いた[AltBeaconを使ってAndroidでiBeaconを検知しよう](http://grandbig.github.io/blog/2015/05/16/altbeacon/)をKotlinで書き直してみました。  

```java
<省略>

// AltBeacon関連のライブラリをimport
import org.altbeacon.beacon.BeaconConsumer;
import org.altbeacon.beacon.BeaconManager;
import org.altbeacon.beacon.BeaconParser;
import org.altbeacon.beacon.Identifier;
import org.altbeacon.beacon.MonitorNotifier;
import org.altbeacon.beacon.Region;

class MainActivity : ActionBarActivity(), BeaconConsumer {
	private val TAG = "AltBeacon Sample"

	// BeaconManagerクラスの変数を定義
	private var beaconManager: BeaconManager? = null

	public val IBEACON_FORMAT = "m:2-3=0215,i:4-19,i:20-21,i:22-23,p:24-24"

	override fun onCreate(savedInstanceState: Bundle?) {
		super.onCreate(savedInstanceState)
		setContentView(R.layout.activity_main)

		// staticメソッドでBeaconManagerクラスのインスタンスを取得
		beaconManager = BeaconManager.getInstanceForApplication(this)
		// BeaconParseをBeaconManagerに設定
		beaconManager!!.beaconParsers.add(BeaconParser().setBeaconLayout(IBEACON_FORMAT))
	}

	override fun onCreateOptionsMenu(menu: Menu): Boolean {
		// Inflate the menu; this adds items to the action bar if it is present.
		menuInflater.inflate(R.menu.menu_main, menu)
		return true
	}

	override fun onOptionsItemSelected(item: MenuItem): Boolean {
		// Handle action bar item clicks here. The action bar will
		// automatically handle clicks on the Home/Up button, so long
		// as you specify a parent activity in AndroidManifest.xml.
		val id = item.itemId

		//noinspection SimplifiableIfStatement
		if (id == R.id.action_settings) {
			return true
		}

		return super.onOptionsItemSelected(item)
	}

	override fun onBeaconServiceConnect() {
		// BeaconManagerクラスのモニタリング通知受取り処理
		beaconManager!!.setMonitorNotifier(object : MonitorNotifier {
			override fun didEnterRegion(region: Region) {
				// 領域進入時に実行
				Log.d(TAG, "didEnterRegion")
			}

			override fun didExitRegion(region: Region) {
				// 領域退出時に実行
				Log.d(TAG, "didExitRegion")
			}

			override fun didDetermineStateForRegion(i: Int, region: Region) {
				// 領域への侵入/退出のステータスが変化したときに実行
				Log.d(TAG, "didDetermineStateForRegion")
			}
		})

		try {
			// UUIDの作成
			val identifier = Identifier.parse("434B4666-E4B2-4B78-8C13-901AFEE3DBB6")
			// モニタリングの開始
			beaconManager!!.startMonitoringBeaconsInRegion(Region("unique-id-001", identifier, null, null))
		} catch (e: RemoteException) {
			// 例外が発生した場合の処理
			e.printStackTrace()
		}
	}

	override fun onPause() {
		super.onPause()
		beaconManager!!.unbind(this)
	}

	override fun onResume() {
		super.onPause()
		beaconManager!!.bind(this)
	}
}
```

こう書いてみると、確かにSwiftに似ていますね。  
気になったのは下記の通りです。  

* Swiftで言うところの`let`がKotlinでは`val`と書くこと  
* Swiftで言うところの`delegate`の設定の仕方とKotlinで言うところの`implements`が同じ  
* Kotlinで変数が`null`のときにメソッドを実行しないよう **!!** が必要  

これからはJavaだけでなく、Kotlinでも書けるように力を蓄えていきたいと思います。  
と言ったところで本日はここまで。  

参考:  

* [SwiftでAndroidアプリを作る!?](http://qiita.com/shoma2da/items/31edef012f224f99323c)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
