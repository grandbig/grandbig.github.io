---
layout: post
title: "Kontakt.ioのSmart Beaconのパラメータを書き換えてみよう！"
date: 2015-07-20 00:01
comments: true
categories: ios iBeacon
---

####Kontakt.ioのSmart Beaconのパラメータの書き換え方法
さて、続きを書いていきます。  
前回、Kontakt.ioのSmart Beaconを検知するところまで書きました。  
本記事では、必要に応じて、パラメータを変更する方法について書きたいと思います。  

手順は[Kontakt.ioの公式ページのBeacon Configuration](http://docs.kontakt.io/ios-sdk/quickstart/#beacon-configuration)に書かれていますが、`/device/update`に引っかかったり、REST APIを叩こうとしたりすることもあるかもしれないので、紹介します。(筆者は初めきちんと読まずにいろいろと実行してしまいました...)  

では、早速書いていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

#####Objective-C編
さて、まずはObjective-Cです。  

```objective-c
// ViewController.m
#import "ViewController.h"
#import "KontaktSDK.h"

@interface ViewController ()<KTKBluetoothManagerDelegate>

@property(strong, nonatomic) KTKBeaconManager *bm;
@property(strong, nonatomic) KTKBeaconDevice *bd;
@property(assign, nonatomic) BOOL flag;

@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];

	_bm = [KTKBeaconManager new];
	_bm.delegate = self;
	_flag = NO;
}

- (void)viewDidAppear:(BOOL)animated {
	[super viewDidAppear:animated];

	// iBeaconデバイスの検索開始
	[_bm startFindingDevices];
}

- (void)viewDidDisappear:(BOOL)animated {
	[super viewDidDisappear:animated];

	// iBeaconデバイスの検索停止
	[_bm stopFindingDevices];
}

- (void)didReceiveMemoryWarning {
	[super didReceiveMemoryWarning];
}

#pragma mark - KTKBluetoothManagerDelegate
- (void)bluetoothManager:(KTKBluetoothManager *)bluetoothManager didChangeDevices:(NSSet *)devices {
	
	if(!_bd) {
		// 利用したいKTKBeaconDeviceが見つかっていない場合
		 NSArray *deviceArray = [devices allObjects];
		 for(id device in deviceArray) {
			KTKBeaconDevice *beacon = device;
			if([beacon.uniqueID isEqualToString:@"****"]) {
				// 利用したいiBeaconを格納
				_bd = beacon;
			}
		 }
	} else {
		// 利用したいKTKBeaconDeviceが見つかった場合
		KTKCharacteristicDescriptor *descriptor;
		descriptor = [_bd characteristicDescriptorWithType:kKTKCharacteristicDescriptorTypeMajor];
		NSError *err;
		// iBeaconデバイスに接続
		BOOL connected = [_bd connectWithPassword:@"****" andError:&err];
		if(connected && !err) {
			// 接続できた場合
			// パラメータ値の取得
			err = [_bd readValueForCharacteristicWithDescriptor:descriptor];
			if(!err) {
				NSString *valueString = [_bd stringForCharacteristicWithDescriptor:descriptor];
				NSLog(@"value: %@", valueString);

				if(!_flag) {
					// パラメータの書き換え処理
					err = [_bd writeString:@"20053" forCharacteristicWithDescriptor:descriptor];
				}
				// iBeaconデバイスから切断
				[_bd disconnect];
			}
		}
	}
}

@end
```

上記処理を実行後、Kontakt.ioの専用アプリで確認すれば、パラメータの値が変更されていることを確認できます。  

#####Swift編
続いて、Swiftのコードも書いてみます。  

```objective-c
// ViewController.swift
import UIKit

class ViewController: UIViewController, KTKBluetoothManagerDelegate {
	
	let bm:KTKBeaconManager = KTKBeaconManager()
	var bd:KTKBeaconDevice? = KTKBeaconDevice()
	var flag:Bool = false

	override func viewDidLoad() {
		super.viewDidLoad()

		self.bm.delegate = self;
	}

	override func viewDidAppear(animated: Bool) {
		super.viewDidAppear(animated)
		
		// iBeaconデバイスの検索開始
		bm.startFindingDevices()
	}

	override func viewDidDisappear(animated: Bool) {
		super.viewDidDisappear(animated)

		// iBeaconデバイスの検索停止
		bm.stopFindingDevices()
	}

	override func didReceiveMemoryWarning() {
		super.didReceiveMemoryWarning()
	}

	func bluetoothManager(bluetoothManager: KTKBluetoothManager!, didChangeDevices devices: Set<NSObject>!) {
		if self.bd?.uniqueID == "****" {
			// 利用したいKTKBeaconDeviceが見つかった場合
			let descriptor:KTKCharacteristicDescriptor = self.bd!.characteristicDescriptorWithType(kKTKCharacteristicDescriptorTypeMajor)
			var err:NSError?
			// iBeaconデバイスに接続
			let connected:Bool = self.bd!.connectWithPassword("****", andError: &err)
			if connected {
				// 接続できた場合
				// パラメータ値の取得
				err = self.bd!.readValueForCharacteristicWithDescriptor(descriptor)
				if err == nil {
					let valueString = self.bd!.stringForCharacteristicWithDescriptor(descriptor)
					println("\(valueString)")
					if self.flag == false {
						// パラメータの書き換え処理
						err = self.bd!.writeString("20053", forCharacteristicWithDescriptor: descriptor)
						self.flag = true;
					}
					// iBeaconデバイスから切断
					self.bd?.disconnect()
				}
			}
		} else {
			// 利用したいKTKBeaconDeviceが見つかっていない場合
			let deviceArray:[AnyObject] = Array(devices)
			for device in deviceArray  {
				let beacon:KTKBeaconDevice = device as! KTKBeaconDevice
				if beacon.uniqueID == "****" {
					// 利用したいiBeaconを格納
					self.bd = beacon
				}
			}
		}
	}
}
```

こちらも専用アプリでパラメータ値が書き換わっていることを確認できました。  
Kontakt.ioの制御特性を活かしたサービスを企画できる可能性を感じました。  
といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
