---
layout: post
title: "OAuth2.0認証の処理を自作しよう！"
date: 2015-06-01 00:56
comments: true
categories: ios oauth2
---

####OAuth2.0認証処理を自分で作ろう！ Objective-C編
本日は、[gtm-oauth](https://code.google.com/p/gtm-oauth2/)や[OAuth2Client](https://github.com/nxtbgthng/OAuth2Client)を使わずにOAuth2.0認証処理を自作してみます。  

OAuth2.0認証を通すためのGoogle設定は[iOSでGoogle OAuth認証がしたい](http://grandbig.github.io/blog/2014/07/13/ios-google-oauth2/)を参照ください。  
筆者が開発したライブラリは単純な下記処理のみ含んでいます。  

* OAuth2.0認証に必要な各種パラメータをKeychain Servicesに保存
* アクセストークンを取得する処理
* リフレッシュトークンから新規アクセストークンを取得する処理

ここで、Keychain Servicesを使うにあたって、[LUKeychainAccess](https://github.com/TheLevelUp/LUKeychainAccess)を利用しています。  
これにより、複雑なKeychain ServicesをNSUserDefaultsの感覚で利用することができます。  
また、Googleへの問い合わせなどのHTTP/HTTPSリクエストに[AFNetworking](https://github.com/AFNetworking/AFNetworking)を利用しています。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####OAuth2.0認証ライブラリの作成
下記にソースをそのまま載せておきます。  

```objective-c

// OAuth2Client.m
#import "OAuth2Client.h"
#import "LUKeychainAccess.h"
#import "AFNetworking.h"

static NSString *callback =  @"http://localhost";
static NSString *visibleactions = @"http://schemas.google.com/AddActivity";

@implementation OAuth2Client

// シングルトンのインスタンス取得
+ (OAuth2Client *)sharedInstance {
	static OAuth2Client* sharedInstance = nil;
	static dispatch_once_t onceToken;
	dispatch_once(&onceToken, ^{
		sharedInstance = [OAuth2Client alloc] init];]
	});

	return sharedInstance;
}

// OAuth2.0認証に必要なパラメータを設定する処理
- (void)setUpOAuth2AccountClientId:(NSString *)clientId clientSecret:(NSString *)clientSecret scope:(NSString *)scope authorizationURL:(NSString *)authorizationURL tokenURL:(NSString *)tokenURL {
	[[LUKeychainAccess standardKeychainAccess] setObject:clientId forKey:@"clientId"];
	[[LUKeychainAccess standardKeychainAccess] setObject:clientSecret forKey:@"clientSecret"];
	[[LUKeychainAccess standardKeychainAccess] setObject:scope forKey:@"scope"];
	[[LUKeychainAccess standardKeychainAccess] setObject:authorizationURL forKey:@"authorizationURL"];
	[[LUKeychainAccess standardKeychainAccess] setObject:tokenURL forKey:@"tokenURL"];
}

// OAuth2.0認証に必要なリクエストを生成する処理
- (void)requestAccessToAccount:(void (^)(NSURL *preparedURL))withPreparedAuthorizationURLHandler {
	NSString *clientId = [[LUKeychainAccess standardKeychainAccess] objectForKey:@"clientId"];
	NSString *scope = [[LUKeychainAccess standardKeychainAccess] objectForKey:@"scope"];
	NSString *authorizationURL = [[LUKeychainAccess standardKeychainAccess] objectForKey:@"authorizationURL"];

	NSString *url = [[NSString stringWithFormat:@"%@?response_type=code&client_id=%@&redirect_uri=%@&scope=%@&data-requestvisibleactions=%@", authorizationURL, clientId, callback, scope, visibleactions];
	withPreparedAuthorizationURLHandler([NSURL URLWithString:url]);
}

// OAuth2.0認証のリダイレクトURIの一致の有無を確認する処理
- (BOOL)checkRedirectURI:(NSURLRequest *)request
{
	// HOSTの取得
	NSString *host = [[request URL] host];
	if ([host isEqualToString:@"localhost"]) {
		return YES;
	} else {
		return NO;
	}
}

// アクセストークンを取得する処理
- (void)getAccessToken:(NSURLRequest *)request completionHandler:(void (^)(NSString *accessToken))completionHandler {
	NSString *host = [[request URL] host];
	if ([host isEqualToString:@"localhost"]) {
		NSString* verifier = nil;
		NSArray* urlParams = [[request URL] query] componentsSeparatedByString:@"&"];
		for (NSString* param in urlParams) {
			NSArray* keyValue = [param componentsSeparatedByString:@"="];
			NSString* key = [keyValue objectAtIndex:0];
			if ([key isEqualToString:@"code"]) {
				verifier = [keyValue objectAtIndex:1];
				break;
			}
		}

		if (verifier) {
			NSString *clientId = [[LUKeychainAccess standardKeychainAccess] objectForKey:@"clientId"];
			NSString *clientSecret = [[LUKeychainAccess standardKeychainAccess] objectForKey:@"clientSecret"];
			NSString *tokenURL = [[LUKeychainAccess standardKeychainAccess] objectForKey:@"tokenURL"];

			// AFHTTPSessionManagerをインスタンス化
			AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
			// サーバエラー時のContent-Typeにtext/plainを許可(成功時にapplication/jsonが必要なので共に追加)
			manager.responseSerializer.acceptableContentTypes = [NSSet setWithObjects:@"text/plain", @"application/json", nil];
			// パラメータの設定
			NSDictionary *data = @{@"code": verifier, @"client_id": clientId, @"client_secret": clientSecret, @"redirect_uri": callback, @"grant_type": @"authorization_code"};

			[manager POST:tokenURL parameters:data success:^(NSURLSessionDataTask *task, id responseObject) {
				// 成功した場合
				if(responseObject && [responseObject count] > 0) {
					NSString *accessToken = responseObject[@"access_token"];
					NSString *refreshToken = responseObject[@"refresh_token"];
					[[LUKeychainAccess standardKeychainAccess] setObject:accessToken forKey:@"accessToken"];
					[[LUKeychainAccess standardKeychainAccess] setObject:refreshToken forKey:@"refreshToken"];

					// 処理が終了したときに実行(アクセストークンを返す)
					completionHandler(accessToken);
				}
			} failure:^(NSURLSessionDataTask *task, NSError *error) {
				// 失敗した場合
				NSError *err;
				NSData *data = [error userInfo] objectForKey:@"com.alamofire.serialization.response.error.data"];
				if(data) {
					// エラーの中身がある場合
					[NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingAllowFragments error:&err];
				}
				// 失敗を返す
				failure(err);
			}];
		}
	} else {
		// HOST名が一致しない場合
	}
}

// リフレッシュトークンから新しいアクセストークンを取得する処理
- (void)getRefreshAccessToken:(void (^)(NSString *accessToken))success failure:(void (^)(NSError *error))failure {
	NSString *clientId = [[LUKeychainAccess standardKeychainAccess] objectForKey:@"clientId"];
	NSString *tokenURL = [[LUKeychainAccess standardKeychainAccess] objectForKey:@"tokenURL"];
	NSString *refreshToken = [[LUKeychainAccess standardKeychainAccess] objectForKey:@"refreshToken"];

	if(clientId && tokenURL && refreshToken) {
		// 必須パラメータがある場合
		// AFHTTPSessionManagerをインスタンス化
		AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
		// サーバエラー時のContent-Typeにtext/plainを許可(成功時にapplication/jsonが必要なので共に追加)
		manager.responseSerializer.acceptableContentTypes = [NSSet setWithObjects:@"text/plain", @"application/json", nil];

		// パラメータの設定
		NSDictionary *data = @{@"client_id": clientId, @"refresh_token": refreshToken, @"grant_type": @"refresh_token"};

		[manager POST:tokenURL parameters:data success:^(NSURLSessionDataTask *task, id responseObject) {
			// 成功した場合
			if(responseObject && [responseObject count] > 0) {
				NSString *accessToken = responseObject[@"access_token"];
				NSString *refreshToken = responseObject[@"refresh_token"];
				[[LUKeychainAccess standardKeychainAccess] setObject:accessToken forKey:@"accessToken"];
				[[LUKeychainAccess standardKeychainAccess] setObject:refreshToken forKey:@"refreshToken"];
				
				// 処理が終了したときに実行(アクセストークンを返す)
				success(accessToken);
			}
		}, failure(NSURLSessionDataTask *task, NSError *error) {
			// 失敗した場合
			NSError *err;
			NSData *data = [error userInfo] objectForKey:@"com.alamofire.serialization.response.error.data"];
			if(data) {
				// エラーの中身がある場合
				[NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingAllowFragments error:&err];
			}
			// 失敗を返す
			failure(err);
		}];
	} else {
		// 必須パラメータがない場合
		// TODO: エラーオブジェクトを生成して返す
		failure(nil);
	}
}

@end

```

```objective-c

// OAuth2Client.h
#import <Foundation/Foundation.h>

@interface OAuth2Client : NSObject

/// シングルトンのインスタンス取得
+ (OAuth2Client *)sharedInstance;

/**
 OAuth2.0認証に必要なパラメータを設定する処理
 @param clientId クライアントID
 @param clientSecret クライアントシークレット
 @param scope アクセス範囲
 @param authorizationURL OAuth2.0 認証先URL
 @param tokenURL OAuth2.0 トークン取得先URL
 */
- (void)setUpOAuth2AccountClientId:(NSString *)clientId clientSecret:(NSString *)clientSecret scope:(NSString *)scope authorizationURL:(NSString *)authorizationURL tokenURL:(NSString *)tokenURL;

/**
 OAuth2.0認証に必要なリクエストを生成する処理
 @param withPreparedAuthorizationURLHandler OAuth2.0認証に必要なリクエストを返すBlock構文
 */
- (void)requestAccessToAccount:(void (^)(NSURL *preparedURL))withPreparedAuthorizationURLHandler;

/**
 OAuth2.0認証のリダイレクトURIの一致の有無を確認する処理
 @param request リクエスト
 @return リダイレクトURIの一致の有無
 */
- (BOOL)checkRedirectURI:(NSURLRequest *)request;

/**
 アクセストークンを取得する処理
 @param request アクセストークンの取得に必要なリクエスト
 @param completionHandler アクセストークンの取得処理が完了したら実行される処理
 */
- (void)getAccessToken:(NSURLRequest *)request completionHandler:(void (^)(NSString *accessToken))completionHandler;

/**
 リフレッシュトークンから新しいアクセストークンを取得する処理
 @param success 処理が成功した場合に実行(返却データはアクセストークン)
 @param failure 処理が失敗した場合に実行(返却データはエラーオブジェクト)
 */
- (void)getRefreshAccessToken:(void (^)(NSString *accessToken))success failure:(void (^)(NSError *error))failure;

```

####呼び出し側のソース
呼び出し側のソースを記載します。  

```objective-c

// ViewController.m
#import "ViewController.h"
#import "OAuth2Client.h"

static NSString *const clientId = @"";
static NSString *const clientSecret = @"";
static NSString *const authorizationURL = @"https://accounts.google.com/o/oauth2/auth";
static NSString *const tokenURL = @"https://accounts.google.com/o/oauth2/token";
static NSString *const scope = @"https://www.googleapis.com/auth/plus.login+https://www.googleapis.com/auth/userinfo.email+https://www.googleapis.com/auth/calendar";

@interface ViewController ()<UIWebViewDelegate>

@property (weak, nonatomic) IBOutlet UIWebView *webView;
@property (strong, nonatomic) NSMutableData *receivedData;
@property (assign, nonatomic) BOOL *isLogin;

@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];

	_webView.delegate = self;
	NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
	if([defaults boolForKey:@"isLogin"]) {
		// 既にOAuth2.0認証実行済みの場合
		[[OAuth2Client sharedInstance] getRefreshAccessToken:^(NSString *accessToken) {
			// 成功した場合
			NSLog(@"%@", accessToken);
		} failure:^(NSError *error) {
			// 失敗した場合
			NSLog(@"%@", error);
		}];
	} else {
		// 初めてOAuth2.0認証を実行する場合
		// OAuth2.0認証に必要な各種パラメータの設定
		[[OAuth2Client sharedInstance] setUpOAuth2AccountClientId:clientId clientSecret:clientSecret scope:scope authorizationURL:authorizationURL tokenURL:tokenURL];
		// OAuth2.0認証リクエスト
		[[OAuth2Client sharedInstance] requestAccessToAccount:^(NSURL *preparedURL) {
			// リクエスト
			[_webView loadRequest:[NSURLRequest requestWithURL: preparedURL]];
		}
	}
}

- (void)didReceiveMemoryWarning {
	[super didReceiveMemoryWarning];
}

- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {

	if([OAuth2Client sharedInstance] checkRedirectURI:request]]) {
		[[OAuth2Client sharedInstance] getAccessToken:request completionHandler:^(NSString *accessToken){
			// 処理が終了したら呼び出される
			if(accessToken.length > 0) {
				NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
				[defaults setBool:YES forKey:@"isLogin"];
				[defaults synchronize];
			}
			[webView removeFromSuperview];
		}];

		return NO;
	}

	return YES;
}

@end

```

これで簡単ですが、OAuth2.0認証の自作ライブラリの開発が完了です。  
必要最低限の機能ですが、十分だと思います。  
(もう少し時間をかけてより良いものを作りたいと思います笑。)  

参考:  

* [Login with Google using OAuth 2.0 for iOS Xcode Objective-C](http://technogerms.com/login-with-google-using-oauth-2-0-for-ios-xcode-objective-c/)
* [OAuth2.0の備忘録的まとめ](http://www.ari-hiro.com/blog/2012/12/30/oauth2-summary)
* [iOSでアプリを削除してもデータを保持する方法](http://qiita.com/asakahara/items/06abbc0209262d0051ef)

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
