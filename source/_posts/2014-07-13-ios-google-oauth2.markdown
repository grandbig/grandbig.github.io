---
layout: post
title: "iOSでGoogle OAuth認証がしたい"
date: 2014-07-13 00:17
comments: true
categories: ios oauth2
---

###iOSネイティブアプリにGoogle OAuth認証機能を実装する方法
Google OAuth認証はWebアプリであれば多くの情報が載っているのですが、なぜかiOSネイティブアプリになると情報が全然見つからない...。  
ということでその方法をまとめてみることにした。  

まず、ネット上で検索してみるとiOSでOAuth認証をするためのライブラリは幾つか作られているようでした。  
その中でも、Googleが提供している**gtm-oauth2**がよく使われているらしい。  
ま、Googleが提供しているのであれば、安心して使えるだろうということで、ここでは**gtm-oauth2**を使った方法で説明します。  

<!--more-->

####設定編
開発に入る前に[Google Console](https://console.developers.google.com/project)で設定する必要があります。  
1: OAuthのClientIDを作成する  
左メニューからAPIS & AUTH > Credentialsを選択します。  
OAuthと書かれた下に**Create new Client ID**というボタンがあります。これをクリックするとポップアップが出てきます。このポップアップの中で必要な選択と入力は下記画像のようになります。  
![Create new Client ID](/images/goauth5.png)  
すると、下記画像のように**CLIENT ID**と**CLIENT SECRET**が表示されます。  
![Client ID & Client Secret](/images/goauth6.png)  

2: Consent Screenを設定する  
次にConsent Screenを設定します。これを設定しないとErrorが発生してしまいます。  
左メニューからAPIS & AUTH > Consent Screenを選択します。  
下記画像のように**EMAIL ADDRESS**と**PRODUCT NAME**の２箇所を入力してSaveボタンをクリックしましょう。  
![Consent Screenの設定](/images/goauth7.png)  

3: 利用するAPIを設定する  
最後に利用したいAPIの設定をONにしておきましょう。  
左メニューからAPIS & AUTH > APIsを選択します。  
下記画像のように利用したいAPIのSTATUSをONにしましょう。  
![APIをONにする](/images/goauth8.png)  

これで設定は完了です。  
続いて開発の説明に入ります。  

####開発編
1: GoogleからソースをGET  
早速、Googleが配布しているソースをもらいに行きます。  
[Googleのgtm-oauth2サイト](https://code.google.com/p/gtm-oauth2/source/checkout)でも書かれているようにsvnを使って、ソースを落とします。  
ソースを保存する場所を決めたら、その配下で下記コマンドを実行。  

```
svn checkout http://gtm-oauth2.googlecode.com/svn/trunk/ gtm-oauth2-read-only
```

もしかしたら、途中でRejectかtemporaryかpermanentか聞かれるかもしれません。筆者はtemporaryにしました。  
ソースを落とした後は[公式ドキュメント](https://code.google.com/p/gtm-oauth2/wiki/Introduction)に書かれている通り、進めていきます。  

2: ダウンロードしたソースを自身のXcodeプロジェクトに追加  
OAuth認証機能を実装したいXcodeプロジェクトを作成します。ここにOAuthに必要なソースを追加します。  
必要なソースは  
・GTMOAuth2Authentication.h/m  
・GTMOAuth2SignIn.h/m  
・GTMHTTPFetcher.h/m  
・GTMOAuth2ViewControllerTouch.h/m  
・GTMOAuth2ViewTouch.xib  
です。  
格納場所は下記画像を見てください。  
![必要なソースの格納場所](/images/goauth1.png)  

ほとんどの人がXcode5.1.1でプロジェクトを作ると思うのですが、デフォルトARC対応プロジェクトとなっていると思います。  
そのため、**コンパイルオプション**と**Other Linker Flags**を設定する必要があります。  
![コンパイルオプションの設定](/images/goauth2.png)  
![Other Linker Flagsの設定](/images/goauth3.png)  

3: Googleからダウンロードしたソースを使うために必要なFrameworkをプロジェクトに追加する  
Security.frameworkとSystemConfiguration.frameworkを追加します。  
![必要なframeworkを追加](/images/goauth4.png)  

4: ソースを書く  
OAuth認証機能を実装するUIViewControllerをプロジェクトに追加しましょう。ここでは名前をViewControllerとします。  
ViewController.mファイルのソース  
```
#import "ViewController.h"
#import "GTMOAuth2Authentication.h"
#import "GTMOAuth2ViewControllerTouch.h"

@interface ViewController ()

@property (nonatomic, retain) GTMOAuth2Authentication *auth;

-(void) startLogin;

@end
static NSString *const kKeychainItemName = @"GOAuthTest";
static NSString *const scope = @"https://www.googleapis.com/auth/calendar";// Calendar APIを利用する場合のscope
static NSString *const clientId = @"xxxxxxxx.apps.googleusercontent.com";// 発行されたClient IDを設定
static NSString *const clientSecret = @"xxxxxxxx";// 発行されたClient Secretを設定
static NSString *const hasLoggedIn = @"hasLoggedInKey";// NSUserDefaultに保存するための文字列

<省略>

- (void)viewDidAppear:(BOOL)animated
{
	// アプリ起動してOAuth認証動作を開始する
	[self startLogin];
}

// OAuth認証の開始
- (void)startLogin
{
	// 既に認証をしたかどうか確認
	NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
	BOOL hasLoggedin = [defaults boolForKey:hasLoggedIn];

	if(hasLoggedin == YES) {
		// 認証したことがある場合
		self.auth = [GTMOAuth2ViewControllerTouch authForGoogleFromKeychainForName:kKeychainItemName
																		  clientID:clientId
																	  clientSecret:clientSecret];
		// アクセストークンの取得
		[self authorizeRequest];
	} else {
		// 認証したことがない場合
		GTMOAuth2ViewControllerTouch *gvc = [[GTMOAuth2ViewControllerTouch alloc] initWithScope:scope
																					   clientID:clientId
																				   clientSecret:clientSecret
																			   keychainItemName:kKeychainItemName
																			  		   delegate:self
																			   finishedSelector:@selector(viewController:finishedWithAuth:error:)
		];
		// 認証画面の表示
		[self presentViewController:gvc animated:YES completion:nil];
	}
}

// 認証後に実行する処理
- (void)viewController:(GTMOAuth2ViewControllerTouch *)viewController
	  finishedWithAuth:(GTMOAuth2Authentication *)auth
	  			 error:(NSError *)error
{
	if(error != nil) {
		// 認証失敗
	} else {
		// 認証成功
		self.auth = auth;
		NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
		[defaults setBool:YES forKey:hasLoggedIn];
		[defaults synchronize];

		// アクセストークンの取得
		[self authorizeRequest];
	}

	// 認証画面を閉じる
	[viewController dismissViewControllerAnimated:YES completion:nil];
}

// アクセストークンの取得処理
- (void)authorizeRequest
{
	NSLog(@"%@", self.auth);
	NSMutableURLRequest *req = [NSMutableURLRequest alloc] initWithURL:self.auth.tokenURL];
	[self.auth authorizeRequest:req completionHandler:^(NSError *error) {
		NSLog(@"%@", self.auth);
	}];
}
```
5. 実機で動作を確認  
ソースが書けたら、実際にアプリを起動して見てみましょう。  
認証画面が表示されます。  
![認証画面](/images/goauth9.png)  

メールアドレスとパスワードを入力すると、認証の許可画面が表示されます。  
![許可画面](/images/goauth10.png)  
許可をすると、開発者の作成する画面に戻ります。  

さて、いかがだったでしょうか？  
今回はGoogle APIの利用まで書いていませんが、OAuthができてしまえば、そんなに難しくはないでしょう。たぶん笑  

参考:  
・[gtm-oauth2でOAuth認証してgoogleのAPIを使う](http://kernhack.hatenablog.com/entry/2013/11/24/164828)  
・[invalid_client no application name - Stack Overflow](http://stackoverflow.com/questions/18677244/error-invalid-client-no-application-name)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
