---
layout: post
title: "SwiftでGCDを使ってみよう！"
date: 2016-01-18 09:26
comments: true
categories: ios swift
---

####GCDとは
GCDとはGrand Central Dispatchの略で、iOSアプリでマルチスレッド処理を実行したいときに利用する手法です。  
書き方は独特ですが、慣れてしまえば、簡単に使いこなすことができると思います。  
今回は筆者が困ったときに使う方法について紹介していきます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####メインスレッドの処理を妨げたくないとき
iOSアプリでは描画はメインスレッドでしか実行できません。  
(もし、それ以外で実行しようとすれば、アプリがダウンしてしまいます。)  

筆者がアプリを開発しているときに、  
検証のため、ログをローカルストレージに保存したい場面がありました。  
ログ保存処理は一時的な検証のためであり、本来のメイン処理を妨げたくはありません。  

ログ保存処理は **描画を含まない処理** であるため、  
別にスレッドを立てて、そちらに任せてしまうことができます。  
これにより、メインスレッドに影響を及ぼすことなく、処理を実行することができます。  

下記にサンプルを載せます。

```objective-c
// ViewController.swift
import UIKit

class ViewController: UIViewController {
	let queue:dispatch_queue_t = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)

	override func viewDidLoad() {
		super.viewDidLoad()

		dispatch_async(queue) {() -> Void in
			// 別スレッドでの処理
			// 例えば、Realmを使った保存処理などを書く(今回は省略します)
		}

		// アラート表示処理
		let alertController:UIAlertController = UIAlertController(title: "確認", message: message, preferredStyle: .Alert)
		let okAction:UIAlertAction = UIAlertAction(title: "OK", style: .Default) { (alert) -> Void in
			// OKを選択したときに実行される処理
		}
		alertController.addAction(okAction)
		presentViewController(alertController, animated: true, completion: nil)
	}

	<省略>
}
```

このように **dispatch_async** 内でRealmへのログ保存処理をすれば、  
別スレッドでの処理となるため、  
ログ保存処理を待たずにアラートの表示処理を実行することができます。  

**dispatch_get_global_queue** とその引数などの詳細は[8.2 Grand Central Dispatch](https://github.com/mixi-inc/iOSTraining/wiki/8.2-Grand-Central-Dispatch)に非常に詳しく書かれているので、ぜひご参照ください。  

####複数の非同期処理の情報で何か処理がしたいとき
例えば、複数種類のWeb APIを叩いて、それぞれから取得した情報を合算して、画面に表示したいとします。  
（別々にAPIを叩いて取得した情報を別々に返せば問題ない場面では当てはまらないので注意）  

そんなときは **dispatch_group_async** と **dispatch_group_notify** を使います。  

下記にサンプルを載せます。  

```objective-c
#import UIKit

class ViewControlelr:UIViewController {

	let queue:dispatch_queue_t = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)
	let group:dispatch_group_t = dispatch_group_create()

	<省略>

	/**
	指定都市の天気情報を取得する処理
	*/
	func getWeather(location:String, closure:(String) -> Void) {
		Alamofire.request(.GET, "http://api.openweathermap.org/data/2.5/weather?APPID=<自身のAPPID>", parameters: 
			["q":location]).response { (request, response, data, error) -> Void in
			if(error == nil) {
				do {
					let dataDict = try NSJSONSerialization.JSONObjectWithData(data!, options: NSJSONReadingOptions.AllowFragments)
					let weatherArray:[AnyObject] = dataDict["weather"] as! [AnyObject]
					let weather:AnyObject = weatherArray[0]
					let description:String = weather["description"] as! String
					closure(description)
				} catch {
					print("例外が発生しました")
				}
			}
		}
	}

	/**
	複数都市の天気情報を取得しログ出力する処理
	*/
	func getMultiLocationWeather() {
		var tokyoWeather:String = "unknown"
		var osakaWeather:String = "unknown"
		var saitamaWeather:String = "unknown"

		// グループに 「+1」
		dispatch_group_enter(self.group)
		dispatch_group_async(self.group, self.queue) { () -> Void in
			self.getWeather("Tokyo, jp", closure: { (weather) -> Void in
				tokyoWeather = weather
				// グループに 「-1」
				dispatch_group_leave(self.group)
			})
		}
		
		// グループに 「+1」
		dispatch_group_enter(self.group)
		dispatch_group_async(self.group, self.queue) { () -> Void in
			self.getWeather("Osaka, jp", closure: { (weather) -> Void in
				osakaWeather = weather
				// グループに 「-1」
				dispatch_group_leave(self.group)
			})
		}
		
		// グループに 「+1」
		dispatch_group_enter(self.group)
		dispatch_group_async(self.group, self.queue) { () -> Void in
			self.getWeather("Saitama, jp", closure: { (weather) -> Void in
				saitamaWeather = weather
				// グループに 「-1」
				dispatch_group_leave(self.group)
			})
		}

		// 全ての処理が終わったら実行される
		dispatch_group_notify(self.group, self.queue) { () -> Void in
			print("Tokyo: \(tokyoWeather), Osaka: \(osakaWeather), Saitama: \(saitamaWeather)")
		}
	}

	<省略>
}
```

以上のように、3つの非同期処理で取得した情報を使って、文章を作成し、ログ出力することができます。  

####非同期処理を同期処理にしたい(排他制御を使いたい)とき
さて、こちらは正直、筆者は利用したいと思ったことがありません。  
恐らく、かなり多機能なアプリで、極力多くの処理を並列に実行したい場合に必要なのかなと思ったりします。  
では、なぜ書くかというと、先日スタックオーバーフローを見て気になったからです。  

因みに、筆者のスタックオーバーフローユーザは[こちら](http://ja.stackoverflow.com/users/8968/takahiro)です。  

[Objective-Cで非同期処理を同期処理にする方法。](https://blog.katty.in/2365)の例をSwiftで書き直します。  

```objective-c
#import UIKit

class ViewController:UIViewController {

	let semaphore:dispatch_semaphore_t = dispatch_semaphore_create(0)
	let queue:dispatch_queue_t = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)
	
	<省略>
	
	func printLog() {
		dispatch_async(queue) { () -> Void in
			for(var i=0; i < 5; i++) {
				sleep(1)
				print("Process: \(i)")
			}
			dispatch_semaphore_signal(self.semaphore)
		}

		print("wait...")
		dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER)
		print("finish")
	}

	<省略>
}
```

実行してみれば、下記のようにログ出力されることがわかるでしょう。  

```objective-c
wait...
Process: 0
Process: 1
Process: 2
Process: 3
Process: 4
finish
```

非同期処理を待ってから、 **finish** とログ出力されていることがわかります。  
この例ではわかりやすさのため、 **dispatch_async** を使って、非同期処理を作り出しています。  

iOSアプリの開発でよく利用する非同期処理と言えば、通信処理ではないでしょうか？  
Swiftでは特に[Alamofire](https://github.com/Alamofire/Alamofire)がよく利用されています。  

Alamofireを使って、上記と似たことをやろうとすると、下記のように書くかもしれません。  

```objective-c
#import UIKit

class ViewController:UIViewController {

	let semaphore:dispatch_semaphore_t = dispatch_semaphore_create(0)
	let queue:dispatch_queue_t = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)
	
	<省略>
	
	func getWeather() {
		Alamofire.request(.GET, "http://api.openweathermap.org/data/2.5/weather?APPID=<自身のAPPID>", parameters:
			[q: "Tokyo, jp"]).response { (request, response, data, error) -> Void in
				if(error == nil) {
					do {
						let dataDict = try NSJSONSerialization.JSONObjectWithData(data!, options: NSJSONReadingOptions.AllowFragments)
						let weatherArray:[AnyObject] = dataDict["weather"] as! [AnyObject]
						let weather:AnyObject = weatherArray[0]
						let description:String = weather["description"] as! String

						dispatch_semaphore_signal(semaphore)
					} catch {
						print("例外が発生しました")
					}
				}
			}
		
		print("wait...")
		dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER)
		print("finish")
	}

	<省略>
}
```

しかしながら、これは実行しても **finish** とログを出力することはありません。  
理由は、**dispatch_semaphore_wait** を実行した後、メインスレッドを停止するのですが、  
`Alamofire`の **response** のクロージャ処理はメインスレッドで実行されるため、  
**dispatch_semaphore_signal** が実行されることがないからです。  

実際に`Alamofire`の`ResponseSerialization.swift` > `Request` クラスを見てみましょう。  

```objective-c
public func response(
	queue queue: dispatch_queue_t? = nil,
	completionHandler: (NSURLRequest?, NSHTTPURLResponse?, NSData?, NSError?) -> Void)
	-> Self
{
	delegate.queue.addOperationWithBlock {
		dispatch_async(queue ?? dispatch_get_main_queue()) {
			completionHandler(self.request, self.response, self.delegate.data, self.delegate.error)
		}
	}

	return self
}
```

となっていることがわかるはずです。  

さて、いかがでしたでしょうか？  
GCDは非常に便利な反面、迂闊に使うと思わぬ落とし穴にハマってしまうことがあります。  
とは言え、  
一度ハマってみないと身につかないのがプログラミングの世界だと思っているので、  
ぜひじゃんじゃん使ってハマってみて頂ければと思います笑  

と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
