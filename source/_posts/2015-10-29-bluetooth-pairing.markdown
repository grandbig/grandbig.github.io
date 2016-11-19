---
layout: post
title: "iOS端末同士でBluetoothペアリングするときの注意点"
date: 2015-10-29 00:45
comments: true
categories: ios
---

####iOS端末同士でのBluetoothペアリング方法について
さて、本日はiOS端末同士でBluetoothペアリングするときの注意点について書きたいと思います。  
筆者は仕様に気づかずにだいぶ苦しめられました笑。  

始めにおさらいの意味も兼ねて、Bluetoothでのペアリングについて書いていきます。  
今回はSwiftで書きます。  

まずはBluetoothの発信側から説明します。

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

```objective-c
import UIKit
import CoreBluetooth

class ViewController: UIViewController, CBPeripheralManagerDelegate {
	
	// 必要なプロパティの宣言
	var pm:CBPeripheralManager!
	let sUUID:CBUUID = CBUUID(string:"47EC089D-2FBB-410D-BEDF-C88730DBBD3A")
	var characteristic:CBMutableCharacteristic!

	override func viewDidLoad() {
		super.viewDidLoad()

		// CBPeripheralManagerの初期化
		self.pm = CBPeripheralManager(delegate: self, queue: nil, options: nil)
	}

	override func didReceiveMemoryWarning() {
		super.didReceiveMemoryWarning()
	}

	// ボタンをタップしたときに実行
	@IBAction func peripheralStartAction(sender: AnyObject) {
		// サービスの追加
		let service = CBMutableService(type: self.sUUID, primary:true)
		self.characteristic = CBMutableCharacteristic(type: sUUID, properties: CBCharacteristicProperties.Read, value: nil, permissions: CBAttributePermissions.ReadEncryptionRequired)
		service.characteristics = [self.characteristic]
		self.pm.addService(service)

		// アドバタイズ開始
		let advertisementData:[String:AnyObject] = [CBAdvertisementDataServiceUUIDsKey: [self.sUUID], CBAdvertisementDataLocalNameKey:"MyBlog"]
		self.pm.startAdvertising(advertisementData)
	}

	// MARK: - CBPeripheralManagerDelegate -
	// Bluetoothの状態が変化したときに通る処理
	func peripheralManagerDidUpdateState(peripheral: CBPeripheralManager) {
		print("state: \(peripheral.state)")
	}

	// アドバタイズの開始結果を取得したときに通る処理
	func peripheralManagerDidStartAdvertising(peripheral: CBPeripheralManager, error: NSError?) {
		if(error != nil) {
			print("アドバタイズ開始失敗。error: \(error)")
			return
		}
														        
		print("アドバタイズ開始成功！")
	}

	// サービスの追加結果を取得したときに通る処理
	func peripheralManager(peripheral: CBPeripheralManager, didAddService service: CBService, error: NSError?) {
		if(error != nil) {
			print("サービスの追加に失敗しました。error: \(error)")
			return
		}
														        
		print("サービス追加成功")
	}

	// Readリクエストを取得したときに通る処理
	func peripheralManager(peripheral: CBPeripheralManager, didReceiveReadRequest request: CBATTRequest) {
					        
		if request.characteristic.UUID.isEqual(self.characteristic.UUID) {
			// プロパティで保持しているキャラクタリスティックへのReadリクエストかどうかを判定
			// CBMutableCharacteristicのvalueをCBATTRequestのvalueにセット
			request.value = self.characteristic.value
																			            
			// リクエストに応答
			self.pm.respondToRequest(request, withResult: CBATTError.Success)
		}
	}
}
```

ここで注目して頂きたいのが、  
`self.characteristic = CBMutableCharacteristic(type: sUUID, properties: CBCharacteristicProperties.Read, value: nil, permissions: CBAttributePermissions.ReadEncryptionRequired)`の部分です。  
`permissions`に`CBAttributePermissions.ReadEncryptionRequired`を設定することで下図のようなペアリングアラートが表示されます。  

![Bluetoothペアリングアラート](/images/bluetooth-pairing-1.png)  

続いて、Bluetoothの受信側の処理を書きます。  

```objective-c
import UIKit
import CoreBluetooth

class ViewController: UIViewController, CBCentralManagerDelegate, CBPeripheralDelegate {
	
	// 必要なプロパティの宣言
	var cm:CBCentralManager!
	var connectPeripheral:CBPeripheral!
	let sUUID:CBUUID = CBUUID(string:"47EC089D-2FBB-410D-BEDF-C88730DBBD3A")

	override func viewDidLoad() {
		super.viewDidLoad()

		// CBCentralManagerの初期化
		self.cm = CBCentralManager(delegate: self, queue: nil)
	}

	override func didReceiveMemoryWarning() {
		super.didReceiveMemoryWarning()
	}

	@IBAction func centralStartAction(sender: AnyObject) {
		// スキャン開始
		self.cm.scanForPeripheralsWithServices([self.sUUID], options: nil)
	}

	// MARK: - CBCentralManagerDelegate -
	// Bluetoothの状態が変化したときに通る処理
	func centralManagerDidUpdateState(central: CBCentralManager) {
		print("state: \(central.state)")
	}

	// 周辺にBLEデバイスが見つかったときに通る処
	func centralManager(central: CBCentralManager, didDiscoverPeripheral peripheral: CBPeripheral, advertisementData: [String : AnyObject], RSSI: NSNumber) {
		if let uuidArray:[CBUUID] = advertisementData["kCBAdvDataServiceUUIDs"] as? [CBUUID] {
			let uuid:CBUUID = uuidArray.first!
			if uuid == self.sUUID {
				// Peripheralに接続
				self.connectPeripheral = peripheral
				self.cm.connectPeripheral(connectPeripheral, options: nil)
			}
		}
	}

	// 接続が成功したときに通る処理
	func centralManager(central: CBCentralManager, didConnectPeripheral peripheral: CBPeripheral) {
		peripheral.delegate = self
		// サービス探索開始
		peripheral.discoverServices([self.sUUID])
	}

	// 接続に失敗したときに通る処理
	func centralManager(central: CBCentralManager, didFailToConnectPeripheral peripheral: CBPeripheral, error: NSError?) {
		print("接続失敗: \(error)")
	}
	
	// MARK: - CBPeripheralDelegate -
	// サービスの探索結果を受け取るときに通る処理
	func peripheral(peripheral: CBPeripheral, didDiscoverServices error: NSError?) {
		let services:[CBService] = peripheral.services!
								        
		for service in services {
			peripheral.discoverCharacteristics([self.sUUID], forService: service)
		}
	}

	// キャラクタリスティックの探索結果を受け取るときに通る処理
	func peripheral(peripheral: CBPeripheral, didDiscoverCharacteristicsForService service: CBService, error: NSError?) {
		let characteristics:[CBCharacteristic] = service.characteristics!
							        
		for characteristic in characteristics {
			print("properties: \(characteristic.properties)")
			peripheral.readValueForCharacteristic(characteristic)
		}
	}

	// データの読込み結果を取得したときに通る処理
	func peripheral(peripheral: CBPeripheral, didUpdateValueForCharacteristic characteristic: CBCharacteristic, error: NSError?) {
		if(error != nil) {
			print("読込みに失敗しました。error: \(error)")
			return
		}
														        
		print("読込みに成功しました。service.uuid: \(characteristic.service.UUID), characteristic uuid: \(characteristic.UUID), value: \(characteristic.value)")
	}
}
```

さあ、これでペアリングアラートが出るはず！ということで2台のiOS端末で試してみました。  
結果、 **『アラートが表示されません！』** でした。  
どこか書き方が間違っているのかと思い、試行錯誤を繰り返したり、ググり続けたりしたところ、理由が判明しました。  
**2台共にiOS8端末かつiCloudに同じApple IDを設定している場合はセキュアと見なされてペアリングアラートが表示されない** ということでした。  

なるほど、出ないわけだ...ということで皆さんも気をつけてみてください。  
本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

