---
layout: post
title: "AltBeaconを使ってAndroidでiBeaconを検知しよう"
date: 2015-05-16 19:12
comments: true
categories: android iBeacon java
---

####AltBeaconを使って手軽にAndroidでiBeaconを検知しよう
本日はAndroidでのiBeacon検知をやってみようと思います。  
自作することも簡単だと聞きますが、Radius Networks社が出している[AltBeacon](https://github.com/AltBeacon/android-beacon-library)が非常に便利なようです。  

何と言ってもiOSのCoreLocation.frameworkと同じ感覚で扱えるというのが強みでしょう。  
(iOSアプリエンジニアにとっては非常に助かります笑)  

さて、実際に実装方法を見て行きましょう。  

実装方法や設定に関しては、[Developers.IO Android x iBeacon – シリーズ –](http://dev.classmethod.jp/series/android-x-ibeacon/)を踏襲させて頂きました。  
めちゃくちゃわかりやすいサイトなので、他のページも参考にすると良いです。 

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

本記事のAndroid Studioのバージョンは1.2となります。  

![Android Studioバージョン](/images/altbeacon_1.png)  

#####AltBeaconのライブラリを使うための準備
AltBeaconライブラリを導入します。  

１. Gradle Scripts > build.gradle(Module: app)の編集
`compile 'org.altbeacon:android-beacon-library:2+'`を`dependencies`に追加

```java
apply plugin: 'com.android.application'

android {

	<省略>

}

dependencies {
	compile fileTree(dir: 'libs', include: ['*.jar'])
	compile 'com.android.support:appcompat-v7:21.0.3'
	compile 'org.altbeacon:android-beacon-library:2+'
}
```

２. app > manifests > AndroidManifest.xmlの編集
Bluetooth関連のpermissionを追加  

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.takahiro.altbeaconsample" >

	<!-- ここを追加 -->
	<uses-permission android:name="android.permission.BLUETOOTH"/>
	<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
	<uses-feature android:name="android.hardware.bluetooth_le" android:required="true"/>

	<application
		
		<省略>

	</application>

</manifest>	
```

#####iBeaconの検知を実装
早速、iBeaconの検知(モニタリング)を実装します。  

```java
package com.example.takahiro.altbeaconsample;

import android.os.RemoteException;
import android.support.v7.app.ActionBarActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.Menu;
import android.view.MenuItem;

// AltBeacon関連のライブラリをimport
import org.altbeacon.beacon.BeaconConsumer;
import org.altbeacon.beacon.BeaconManager;
import org.altbeacon.beacon.BeaconParser;
import org.altbeacon.beacon.Identifier;
import org.altbeacon.beacon.MonitorNotifier;
import org.altbeacon.beacon.Region;

public class MainActivity extends ActionBarActivity implements BeaconConsumer {
	// BeaconConsumerインターフェースを実装
	private static String TAG = "AltBeacon Sample";
	// BeaconManagerクラスの変数を定義
	private BeaconManager beaconManager;

	// iBeaconのデータを認識するためのParserフォーマット
	public static final String IBEACON_FORMAT = "m:2-3=0215,i:4-19,i:20-21,i:22-23,p:24-24";

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		// staticメソッドでBeaconManagerクラスのインスタンスを取得
		beaconManager = BeaconManager.getInstanceForApplication(this);
		// BeaconParseをBeaconManagerに設定
		beaconManager.getBeaconParsers().add(new BeaconParser().setBeaconLayout(IBEACON_FORMAT));
	}

	@Override
	public boolean onCreateOptionsMenu(Menu menu) {
		// Inflate the menu; this adds items to the action bar if it is present.
		getMenuInflater().inflate(R.menu.menu_main, menu);
		return true;
	}

	@Override
	public boolean onOptionsItemSelected(MenuItem item) {
		// Handle action bar item clicks here. The action bar will
		// automatically handle clicks on the Home/Up button, so long
		// as you specify a parent activity in AndroidManifest.xml.
		int id = item.getItemId();

		//noinspection SimplifiableIfStatement
		if(id == R.id.action_settings) {
			return true;
		}

		return super.onOptionsItemSelected(item);
	}

	@Override
	public void onBeaconServiceConnect() {
		// BeaconManagerクラスのモニタリング通知受取り処理
		beaconManager.setMonitorNotifier(new MonitorNotifier() {
			@Override
			public void didEnterRegion(Region region) {
				// 領域進入時に実行
				Log.d(TAG, "didEnterRegion");
			}

			@Override
			public void didExitRegion(Region region) {
				// 領域退出時に実行
				Log.d(TAG, "didExitRegion");
			}

			@Override
			public void didDetermineStateForRegion(int i, Region region) {
				// 領域への侵入/退出のステータスが変化したときに実行
				Log.d(TAG, "didDetermineStateForRegion");
			}
		});

		try {
			// UUIDの作成
			Identifier identifier = Identifier.parse("434B4666-E4B2-4B78-8C13-901AFEE3DBB6");
			// モニタリングの開始
			beaconManager.startMonitoringBeaconsInRegion(new Region("unique-id-001", identifier, null, null));
		} catch(RemoteException e) {
			// 例外が発生した場合の処理
			e.printStackTrace();
		}
	}

	@Override
	protected void onPause() {
		super.onPause();
		beaconManager.unbind(this);
	}

	@Override
	protected void onResume() {
		super.onPause();
		beaconManager.bind(this);
	}
}
```

ポイントは下記です。  

* iBeacon検知のためにBeaconParserを定義して、BeaconLayoutに設定
* `startMonitoringBeaconsInRegion`する際の引数であるRegionへの定義
各引数は`Region("監視対象ID", "UUID", "major", "minor")`のようになっています。  
**監視対象ID** さえ設定すれば、iBeaconを検知可能です。UUID, major, minorを設定すると、該当するiBeaconのみ検知するようになります。  

#####iBeaconで距離計測
次にiBeaconの距離計測をしてみましょう。  
今回は領域観測をして、領域に入ったタイミングで距離計測を開始し、領域を出たタイミングで距離計測を停止します。  

```java

<省略>

@Override
public void onBeaconServiceConnect() {
	// BeaconManagerクラスのモニタリング設定
	beaconManager.setMonitorNotifier(new MonitorNotifier() {
		@Override
		public void didEnterRegion(Region region) {
			// 領域侵入時に実行
			Log.d(TAG, "didEnterRegion");

			try {
				// レンジングの開始
				beaconManager.startRangingBeaconsInRegion(new Region("unique-id-001", identifier, null, null));
			} catch(RemoteException e) {
				// 例外が発生した場合
				e.printStackTrace();
			}
		}

		@Override
		public void didExitRegion(Region region) {
			// 領域退出時に実行
			Log.d(TAG, "didExitRegion");

			try {
				// レンジングの停止
				beaconManager.stopRangingBeaconsInRegion(new Region("unique-id-001", identifier, null, null));
			} catch(RemoteException e) {
				// 例外が発生した場合
				e.printStackTrace();
			}
		}
	}
	
	// BeaconManagerクラスのレンジング設定
	beaconManager.setRangeNotifier(new RangeNotifier() {
		@Override
		public void didRangeBeaconsInRegion(Collection<Beacon> beacons, Region region) {
			// 検出したビーコンの情報を全部Logに書き出す
			for(Beacon beacon : beacons) {
				Log.d(TAG, "UUID:" + beacon.getId1() + ", major:" + beacon.getId2() + ", minor:" + beacon.getId3() + ", Distance:" + beacon.getDistance() + ",RSSI" + beacon.getRssi() + ", TxPower" + beacon.getTxPower());
			}
		}
	}

	try {
		// モニタリングの開始
		beaconManager.startMonitoringBeaconsInRegion(new Region("unique-id-001", identifier, null, null));
	} catch(RemoteException e) {
		e.printStackTrace();
	}
}

<省略>

```

AltBeaconライブラリはだいぶ使いやすいですね！  
iBeaconはiOS先行でサービスを始めることが多いと思うので、iOSの仕様に合わせてプログラミングを組めるのはエンジニアとしてありがたいですね。  
(Androidユーザが多いとは言えない日本にとっては、Android特有の機能を設けることは工数的に優先すべきではなかったりしますしね。趣味として取り入れたい気持ちはありますが...。)

と言ったところで本日はここまで。  

[本記事のレポジトリはこちら](https://github.com/grandbig/altBeaconSample)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
