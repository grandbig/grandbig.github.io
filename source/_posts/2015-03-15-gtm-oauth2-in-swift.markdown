---
layout: post
title: "SwiftでGoogle OAuth2認証をしよう！"
date: 2015-03-15 22:20
comments: true
categories: ios oauth2 swift
---

####Swift版 OAuth2認証
本日は[iOSでGoogle OAuth認証がしたい](http://grandbig.github.io/blog/2014/07/13/ios-google-oauth2/)のSwift版を紹介したいと思います。  
Google Play Consoleでの設定やGoogleから配布されているOAuth2認証に必要なソースのダウンロード方法については上記の記事に任せるとして、具体的なソース部分を紹介します。  

では早速、Objective-CのソースをSwiftで書きなおしたものを下記に載せます。

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

```objective-c
// ViewController.swift
import UIKit

class ViewController: UIViewController {

	var auth:GTMOAuth2Authentication!
	let kKeychainItemName:NSString! = "GOAuthTest"
	let scope:NSString! = "https://www.googleapis.com/auth/userinfo.profile https://www.googleapis.com/auth/calendar"
	let clientId:NSString! = "Googleから発行されたクライアントID"
	let clientSecret:NSString! = "Googleから発行されたクライアントシークレット"
	let hasLoggedIn:NSString! = "hasLoggedInKey"

	override func viewDidLoad() {
		super.viewDidLoad()
	}

	override func viewDidAppear(animated: Bool) {
		self.startLogIn()
	}

	override func didReceiveMemoryWarning() {
		super.didReceiveMemoryWarning()
	}

	func startLogIn() {
		var defaults:NSUserDefaults? = NSUserDefaults.standardUserDefaults()
		var hasLoggedInFlag:Bool! = defaults?.boolForKey(hasLoggedIn)

		if(hasLoggedInFlag == true) {
			// 認証したことがある場合
			self.auth = GTMOAuth2ViewControllerTouch.authForGoogleFromKeychainForName("Google", clientID: clientId, clientSecret: clientSecret)

			// アクセストークンの取得
			self.authorizeRequest()
		} else {
			// 認証したことがない場合
			var gvc:GTMOAuth2ViewControllerTouch! = GTMOAuth2ViewControllerTouch(scope: scope, clientID: clientId, clientSecret: clientSecret, keychainItemName: "Google", delegate: self, finishedSelector: "viewController:finishedWithAuth:error:")
			self.presentViewController(gvc, animated: true, completion: nil)
		}
	}

	func viewController(viewController:GTMOAuth2ViewControllerTouch!, finishedWithAuth:GTMOAuth2Authentication!, error:NSError?) {
		if(error != nil) {
			// 認証失敗
		} else {
			// 認証成功
			self.auth = finishedWithAuth
			var defaults = NSUserDefaults.standardUserDefaults()
			defaults.setBool(true, forKey: hasLoggedIn)
			defaults.synchronize()

			// アクセストークンの取得
			self.authorizeRequest()
		}

		// 認証画面を閉じる
		viewController.dismissViewControllerAnimated(true, completion: nil)
	}

	func authorizeRequest() {
		println(self.auth)
		var req:NSMutableURLRequest! = NSMutableURLRequest(URL: self.auth.tokenURL)
		self.auth.authorizeRequest(req, completionHandler: { (error) -> Void in
			println(self.auth)
		})
	}
}
```

今回のソースは[GitHub](https://github.com/grandbig/gtm-oauth2-swift)にアップしています。  
興味があればどしどし使ってみてください。  
ということで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
