---
layout: post
title: "Google Sign-In SDK for iOSを使ってみよう！"
date: 2015-09-13 02:34
comments: true
categories: ios, Google
---

####Google Sign-in SDKを使ったOAuth認証をやってみよう！
さて、本日はGoogle Sign-Inを利用したOAuth認証をやってみようと思います。  
最近、[Google Developers Console](https://console.developers.google.com)の画面が完全に一新されていました。  
以前であれば、Google Developers ConsoleにOAuth2.0のiOSのクライアントIDを設定したときに、クライアントIDだけでなく、クライアントシークレットなども表示されていました。  
それが、 **クライアントID** と **iOSのURLスキーム** のみになっていました。  
これは完全にGoogleがGoogle Sign-Inを推奨している証であろうということで、早速使い方を見ていきたいと思います。

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####Google Developers ConsoleでOAuth認証に必要な項目を登録しよう！
さて、実際にGoogle Sign-Inを始める前に、やらなければならないことがあります。  
それは[Google Developers Console](https://console.developers.google.com)での登録作業です。  
冒頭に述べたようにサイトのデザインや表示項目なども変わっているので、1つ1つ見ていきましょう。  

まず、OAuth2.0の同意画面を作成しましょう。  
因みに、筆者は既に登録済みのOAuth2.0クライアントIDがあったため、下図のように表示されます。  
**OAuth同意画面** というボタンをクリックして、OAuth同意画面の作成ページに移りましょう。  
![認証情報の確認ページ](/images/google-sign-in-5.png)  

OAuth同意画面では **メールアドレス** と **サービス名** を入力しましょう。(他は必要に応じて入力してください。)  
![OAuth2.0の同意画面を作成](/images/google-sign-in-6.png)  

OAuth同意画面が作成できたら、認証情報を追加しましょう。  
下図のように **OAuth2.0クライアントID** を選択してください。  
![OAuth2.0クライアントIDを作成](/images/google-sign-in-7.png)  

今回はiOS用なので、下図の手順でクライアントIDを作成しましょう。  
![OAuth2.0クライアントIDを作成②](/images/google-sign-in-8.png)  

これでクライアントIDが作成されたと思います。  
作成したクライアントIDを選択すれば、下図のような情報が得られるはずです。  
 ![Google Developer ConsoleのOAuth2.0 for iOS](/images/google-sign-in-1.png)  

####Google Sign-in SDK for iOSを実装しよう
当然のことながら、基本的なことは[Googleの公式サイト](https://developers.google.com/identity/sign-in/ios/start-integrating)に書かれています。  
とは言え、英語ですし、案外躓くところもあったりするので、順を追って説明していきます。

まずは、[Google Sign-In for iOSのガイドページ](https://developers.google.com/identity/sign-in/ios/start)にアクセスしましょう。  
そうすると、順番に説明されていることがわかります。  
①はテストコードを手に入れるだけなので、自分のプロジェクトに組み込むなら②からはじめましょう。  
![GET A CONFIGURATION FILE](/images/google-sign-in-9.png)  

そうすると、下図のようなページに遷移します。  
![Enable Google services for your app](/images/google-sign-in-10.png)  

ここで **App name** にGoogle Developer Consoleのプロジェクト名を **『選択』** します。  
注意点として、App nameに記載されているデフォルト名を削除すると、選択可能なプロジェクト名が表示されます。  
App nameが入力されれば、 **iOS Bundle ID** も選択可能になります。  

必要事項を入力した後はChoose and configure servicesボタンを選択しましょう。  
![Choose and configure services](/images/google-sign-in-11.png)  

結果、画面が遷移します。  
![Configuration Fileを作成しよう](/images/google-sign-in-12.png)  

画面下の方のGenerate configuration filesボタンをクリックしましょう。  
![Configuration Fileを作成しよう](/images/google-sign-in-13.png)  

plistファイルのダウンロード画面が表示されるので、ダウンロードしましょう。  
![plistファイルをダウンロード](/images/google-sign-in-14.png)  

必要なファイルが揃ったので、 **Google Sign-In for iOS ライブラリ** を自分のプロジェクトに導入しましょう。  
Podfileを作成して、`pod 'Google/SignIn`と記載します。あとは`pod install`でOK  

さて`xcworkspace`ファイルが作成されているはずなので、Xcodeで開きましょう。  
先ほどダウンロードした`GoogleService-Info.plist`ファイルをプロジェクトに組み込みましょう。  
ファイルを選択して、Xcodeプロジェクトにドラッグ＆ドロップでOKです。  

続いて、`GoogleService-Info.plist`ファイルに記載された **REVERSED_CLIENT_ID** と **BUNDLE_ID** をURL schemesとして設定する必要があります。  
TARGETS > info > URL TypesでURL Typeを2つ追加しましょう。  
![URL Typesを追加](/images/google-sign-in-15.png)  

これでソースコードを書くまでの準備が整いました。  
次にソースコードを書いていきます。  

```objective-c
// AppDelegate.h
#import <UIKit/UIKit.h>
#import <Google/SignIn.h>

@interface AppDelegate : UIResponder <UIApplicationDelegate, GIDSignInDelegate>

@property (strong, nonatomic) UIWindow *window;

@end
```

```objective-c
// AppDelegate.m
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
	// Override point for customization after application launch.
			    
	NSError* configureError;
	[[GGLContext sharedInstance] configureWithError: &configureError];
	NSAssert(!configureError, @"Error configuring Google services: %@", configureError);
						    
	[GIDSignIn sharedInstance].delegate = self;
								    
	return YES;
}

<省略>

- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation {
	    
	return [[GIDSignIn sharedInstance] handleURL:url sourceApplication:sourceApplication annotation:annotation];
}

// Google Sign-inに成功した場合に呼び出される処理
- (void)signIn:(GIDSignIn *)signIn didSignInForUser:(GIDGoogleUser *)user withError:(NSError *)error {
	// Perform any operations on signed in user here.
	NSString *userId = user.userID;                  // For client-side use only!
	NSString *idToken = user.authentication.idToken; // Safe to send to the server
	NSString *name = user.profile.name;
	NSString *email = user.profile.email;
						    
	NSLog(@"userId: %@, idToken: %@, name: %@, email: %@", userId, idToken, name, email);
}

// Google Sign-inに失敗した場合に呼び出される処理
- (void)signIn:(GIDSignIn *)signIn didDisconnectWithUser:(GIDGoogleUser *)user withError:(NSError *)error {
	// Perform any operations when the user disconnects from app here.
	NSLog(@"error: %@", error);
}

```

```objective-c
// ViewController.h
#import <UIKit/UIKit.h>
#import <Google/SignIn.h>

@interface ViewController : UIViewController <GIDSignInUIDelegate>

@property (weak, nonatomic) IBOutlet GIDSignInButton *signInButton;

@end

```

```objective-c
// ViewController.m
- (void)viewDidLoad {
	[super viewDidLoad];
	// Do any additional setup after loading the view, typically from a nib.
			    
	[GIDSignIn sharedInstance].uiDelegate = self;
	// 任意のスコープを設定
	[GIDSignIn sharedInstance].scopes = @[@"https://www.googleapis.com/auth/calendar", @"https://www.googleapis.com/auth/drive"];
	NSLog(@"scope: %@", [GIDSignIn sharedInstance].scopes);
}

// エラーが発生した場合に呼び出される処理
- (void)signInWillDispatch:(GIDSignIn *)signIn error:(NSError *)error {
	NSLog(@"error: %@", error);
}

// Google OAuth2認証画面を開始するときに実行される処理
- (void)signIn:(GIDSignIn *)signIn presentViewController:(UIViewController *)viewController {
	// OAuth2認証画面を表示する処理
	[self presentViewController:viewController animated:YES completion:nil];
}

// Google OAuth2認証が完了したら実行される処理
- (void)signIn:(GIDSignIn *)signIn dismissViewController:(UIViewController *)viewController {
	// OAuth2認証の画面を閉じる処理
	[self dismissViewControllerAnimated:YES completion:nil];
}

// Google Sign-inボタンをクリックした場合の処理(Storyboardで設定すること)
- (IBAction)didTapSignIn:(id)sender {
	// Google Sign-inを実行する処理
	[[GIDSignIn sharedInstance] signIn];
}
```

上記を実行することで下記のようにOAuth2認証ができました。  

![Google OAuth2認証開始画面](/images/google-sign-in-2.png)  
![Google OAuth2認証の認証許可確認画面](/images/google-sign-in-3.png)  

**Google+アプリ** がインストールされている場合はSafariではなく **Google+アプリ** が起動します。  

さて、いかがだったでしょうか？  
今回はひとまず、ここまでにしますが、今後はOAuth2認証後にGoogle Calendar APIやGoogle Driveの画面表示をしてみたいと思います。  
といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
