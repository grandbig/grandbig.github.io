---
layout: post
title: "SwiftでFacebookログインを始めよう！"
date: 2016-05-14 15:30
comments: true
categories: ios swift
---

####Facebookログインの実装方法
公式ページの[facebook for developers](https://developers.facebook.com/)を見るのが、一番良いと思いますが、  
あちこちページを飛ばされるので、順を追って説明することにします。  
※ 本記事で利用しているFacebook SDKは **v4.11.0** です。  

**1．Facebookログインを選択する**  

![Facebookログインを選択](/images/facebook_login_1.png)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

**2．Get Startedを選択する**  

![Get Startedを選択](/images/facebook_login_2.png)  

**3．iOSを選択する**  

![iOSを選択](/images/facebook_login_3.png)  

**4．利用手順の概要を読む**  

![利用手順の概要を読む](/images/facebook_login_4.png)  

**5．Facebookアプリを作成する**  

![Facebookアプリを作成](/images/facebook_login_5.png)  

※ CocoaPodsでFacebook SDKを取り込むので、ダウンロードはしません。  

「新しいアプリを作成」を選択すると、アプリ作成画面が表示されるため、任意の値を入力してください。  

![任意の値を入力](/images/facebook_login_6.png)  

**6．作成したFacebookアプリにiOSアプリ用の設定を追加する**  

![一連のiOSアプリ用設定手順](/images/facebook_login_7.png)  

![追加ボタンを選択](/images/facebook_login_8.png)  

![iOSを選択](/images/facebook_login_9.png)  

追加された設定項目に値を入力して、「変更を保存」ボタンを選択します。  

* バンドルIDを入力  
* シングルサインオンボタンをONに変更  

![設定項目に値を入力](/images/facebook_login_10.png)  

続いて、詳細設定でネイティブアプリでの利用を許可します。  

![詳細設定](/images/facebook_login_18.png)  

**7．Xcodeプロジェクトを用意する**  

当然のことながら、Xcodeプロジェクトを作成します。  

**8．`Podfile`を作成する**  

```objective-c
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '8.0'

# Facebook
pod 'FBSDKCoreKit'
pod 'FBSDKLoginKit'
pod 'FBSDKShareKit'
```

**9．`pod install`を実行する**  

**10．Xcodeプロジェクトに必要な設定を追加**  

必要な設定値を見るために、  
下図のようにFacebookアプリページのiOS設定項目上の **クイックスタート** を選択します。  

![クイックスタートを選択](/images/facebook_login_11.png)  

すると、下図のページに遷移します。  

![クイックスタートページ](/images/facebook_login_12.png)  

この中で重要なのが **Configure your info.plist** です。  
ここに書かれた設定値をXcodeに追加します。  

![Configure your info.plist](/images/facebook_login_13.png)  

![Xcodeに設定値を追加](/images/facebook_login_14.png)  

**11．XcodeプロジェクトにiOS9 SDK対応の設定を追加**  

iOS9 SDKよりHTTP通信が非推奨となったため、ATS設定を追加する必要があります。  

![XcodeにATS設定を追加](/images/facebook_login_15.png)  

**12．Facebookアプリをホワイトリストに追加する**  

![ホワイトリストへの追加](/images/facebook_login_16.png)  

**13．Bridge-Headerファイルを追加**  

Swift実装なので、Bridge-Headerファイルを追加します。  

```objective-c
// FacebookLoginSample-Bridging-Header.h

#ifndef FacebookLoginSample_Bridging_Header_h
#define FacebookLoginSample_Bridging_Header_h

#import <FBSDKCoreKit/FBSDKCoreKit.h>
#import <FBSDKLoginKit/FBSDKLoginKit.h>


#endif /* FacebookLoginSample_Bridging_Header_h */
```

**14．`AppDelegate.swift`に`FBSDKApplicationDelegate`への接続処理を追加**  

```objective-c
// AppDelegate.swift

func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {

	FBSDKApplicationDelegate.sharedInstance().application(application, didFinishLaunchingWithOptions: launchOptions)
	return true;
}

func application(application: UIApplication, openURL url: NSURL, sourceApplication: String?, annotation: AnyObject) -> Bool {

	return FBSDKApplicationDelegate.sharedInstance().application(application,
																 openURL: url,
																 sourceApplication: sourceApplication,
																 annotation: annotation)
}
```

**15．Facebook SDKが機能しているか確認**  

Facebook SDKを搭載したつもりでも、一切機能していないと困りますよね？  
ということでログ収集機能を使ってTestできたりします。  
様々なログ収集が可能なようですが、公式ページにあるように最も簡単なアプリ起動ログ収集を実装します。  

```objective-c
// AppDelegate.swift
func applicationDidBecomeActive(application: UIApplication) {
	FBSDKAppEvents.activateApp()
}
```

[アプリ用Analyticsダッシュボード](https://www.facebook.com/analytics?__aref_src=devsite&__aref_id=docs_ios_getting_started)に遷移します。  
ログを確認したいアプリ > アクティビティー > イベントを選択します。  
すると「App Launches」の件数を確認することができます。  

**16．ログインボタンを追加**  

公式ではソースコードから追加しているので、その想定で書いていますが、もちろんStoryboardも利用できます。  

```objective-c
// ViewController.swift
import UIKit

class ViewController: UIViewController, FBSDKLoginButtonDelegate {

	override func viewDidLoad() {
		super.viewDidLoad()

		let loginBtn : FBSDKLoginButton = FBSDKLoginButton()
		self.view.addSubview(loginBtn)
		loginBtn.center = self.view.center
		loginBtn.readPermissions = ["public_profile", "email", "user_friends"]
		loginBtn.delegate = self
	}
}

func loginButton(loginButton: FBSDKLoginButton!, didCompleteWithResult result: FBSDKLoginManagerLoginResult!, error: NSError!) {
	
	print("ログイン処理を実行")

	if (error != nil) {
		// エラーが発生した場合
		print("Process Error !")
	} else if result.isCancelled {
		// ログインをキャンセルした場合
		print("User Cancelled")
	} else {
		// その他
		print("Login Succeeded")
	}
}

func loginButtonDidLogOut(loginButton: FBSDKLoginButton!) {

	print("ログアウト処理を実行")
}
```

結果、下図のような遷移が可能となります。  

![Facebookログイン](/images/facebook_login_17.png)  

####Facebookログインボタンのカスタム化
因みに、Facebookログインボタンをカスタム化する方法を説明します。  
折角なので、Storyboardでの方法として載せておきましょう。  

**1．Storyboardにボタンを配置する**  

![Storyboardにボタンを配置](/images/facebook_login_19.png)  

**2．StoryboardからViewControllerにボタンアクションを接続する**  

![ボタンアクションの接続](/images/facebook_login_20.png)  
※ 上記キャプチャでは既に下記3の処理を書いてあるけど悪しからず...  

**3．ログインアクションを追加**  

```objective-c
// ViewController.swift
@IBAction func loginByFacebookID(sender: AnyObject) {
	let loginManager: FBSDKLoginManager = FBSDKLoginManager()
	loginManager.logInWithReadPermissions(["public_profile", "email", "user_friends"], fromViewController: self) { (result, error) in
		if (error != nil) {
			// エラーが発生した場合
			print("Process error")
		} else if result.isCancelled {
			// ログインをキャンセルした場合
			print("Cancelled")
		} else {
			// その他
			print("Login Succeeded")
		}
	}
}
```

さて、いかがでしたでしょうか？  
Facebook SDKも徐々にバージョンアップしていますので、なかなか最新のライブラリに対応した記事がないこともあるでしょう。  
そんなときに本記事を少しでも参考にしてもらえればと思います。  
ということで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

