---
layout: post
title: "AndroidでBackgroundでiBeaconを検知しよう！"
date: 2015-06-06 22:00
comments: true
categories: android iBeacon java
---

####AltBeaconでiBeacon検知(アプリBG起動編)
さて、以前、[AltBeaconを使ってAndroidでiBeaconを検知しよう](http://grandbig.github.io/blog/2015/05/16/altbeacon/)で基本的な使い方を説明させて頂きました。  
今回はアプリをBackgroundで起動しているときにもiBeaconを検知できるようにしてみようと思います。  
Background起動だけであれば、Applicationクラスを使うことで実現できますが、  
Serviceクラスを使えば、アプリをBackgroundから削除したとしても動作させることができます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

では早速、ソースコードを見て行きましょう。  


```java
// BeaconServiceクラス
// Serviceを拡張し、BootstrapNotifierをインターフェースとしたBeaconServiceクラス
public class BeaconService extends Service implements BootstrapNotifier {

	public static final String TAG = org.altbeacon.beacon.service.BeaconService.class.getSimpleName();
	
	// iBeaconのデータを認識するためのParserフォーマット
	public static final String IBEACON_FORMAT = "m:2-3=0215,i:4-19,i:20-21,i:22-23,p:24-24";
	// BGで監視するiBeacon領域
	private RegionBootstrap regionBootstrap;
	// iBeacon検知用のマネージャー
	private BeaconManager beaconManager;
	// UUID設定用
	private Identifier identifier;
	// iBeacon領域
	private Region region;
	// 監視するiBeacon領域の名前
	private String beaconName;

	@Override
	public void onCreate() {
		super.onCreate();

		// iBeaconのデータを受信できるようにParserを設定
		beaconManager = BeaconManager.getInstanceForApplication(this);
		beaconManager.getBeaconParsers().add(new BeaconParser().setBeaconLayout(IBEACON_FORMAT));
		// BGでiBeacon領域を監視(モニタリング)するスキャン間隔を設定
		beaconManager.setBackgroundBetweenScanPeriod(1000);

		// UUIDの作成
		identifier = Identifier.parse("A56BA1E1-C06E-4C08-8467-DB6F5BD04486");
		// Beacon名の作成
		beaconName = "MyBeacon-000206C6";
		// major, minorの指定はしない
		region = new Region(beaconName, identifier, null, null);
		regionBootstrap = new RegionBootstrap(this, region);

		beaconManager.setRangeNotifier(new RangeNotifier() {
			@Override
			public void didRangeBeaconsInRegion(Collection<Beacon> beacons, Region region) {
				// 検出したビーコンの情報を全部Logに書き出す
				for(Beacon beacon : beacons) {
					Log.d(TAG, "UUID:" + beacon.getId1() + ", major:" + beacon.getId2() + ", minor:" + beacon.getId3() + ", Distance:" + beacon.getDistance() + ",RSSI" + beacon.getRssi() + ", TxPower" + beacon.getTxPower());
				}
			}
		});
	}

	@Override
	public int onStartCommand(Intent intent, int flags, int startId) {
		return super.onStartCommand(intent, flags, startId);
	}

	@Override
	public void onDestroy() {
		super.onDestroy();
	}

	@Override
	public IBinder onBind(Intent intent) {
		return null;
	}

	@Override
	public void didEnterRegion(Region region) {
		// 領域侵入
		Log.d(TAG, "Enter Region");
		// アプリをFG起動させる
		Intent intent = new Intent(this, MainActivity.class);
		intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
		startActivity(intent);

		try {
			// レンジング開始
			beaconManager.startRangingBeaconsInRegion(region);
		} catch(RemoteException e) {
			// 例外が発生した場合
			e.printStackTrace();
		}
	}

	@Override
	public void didExitRegion(Region region) {
		// 領域退出
		Log.d(TAG, "Exit Region");
		try {
			// レンジング停止
			beaconManager.stopRangingBeaconsInRegion(region);
		} catch() {
			// 例外が発生した場合
			e.printStackTrace();
		}
	}

	@Override
	public void didDetermineStateForRegion(int i, Region region) {
		// 領域に対する状態が変化
		Log.d(TAG, "Determine State: " + i);
	}
}
```

**領域進入時にアプリをFG起動** させているので、アプリがFG起動したらiBeacon領域に侵入したと言えます。  
さあ、これでiOSとほぼ同じ挙動を実現できました。  

蛇足かもしれませんが、`MainActivity.java`でサービスを起動する部分も書いておきます。

```java
// MainActivity.java

public class MainActivity extends ActionBarActivity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		// サービス起動
		startService(new Intent(MainActivity.this, BeaconService.class));
	}

	<省略>
}

```


ソースコードは[こちら](https://github.com/grandbig/altBeaconSample/tree/service)から。  
serviceブランチを参照ください。  

といったところで本日はここまで。  

参考:  
[[Android][iBeacon] Android Beacon Library パラっと解説 その5 [バックグラウンド領域監視]](http://dev.classmethod.jp/smartphone/android-beacon-library-introduction-5/)

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
