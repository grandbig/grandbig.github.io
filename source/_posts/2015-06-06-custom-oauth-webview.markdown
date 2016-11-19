---
layout: post
title: "OAuth2.0認証をUIWebView&WKWebViewで実施しよう！"
date: 2015-06-06 23:38
comments: true
categories: ios webview WKWebView
---

####最適なWebViewを使って、OAuth2.0認証をしよう！
本日は、iOSのバージョン毎に最適なWebViewを使ってOAuth2.0認証を実施してみたいと思います。  
やはり、iOS8であればWKWebViewを使いたいですし、iOS7ではUIWebViewしか利用できないという現実があります。  

WKWebViewとUIWebViewの使い分け処理に関しては、[let UIWebView as WKWebView](http://techblog.yahoo.co.jp/ios/let-uiwebview-as-wkwebview/)を参考にさせて頂きました。  
(あえてObjective-Cで書いています。)  

まずは、UIWebViewとWKWebViewの使い分け部分を説明していきます。  

共通インターフェースとして`WKWebViewProtocol.h`を作成します。  
共通化したい処理を書き出しています。  

```objective-c
// WKWebViewProtocol.h

#import <UIKit/UIKit.h>
#import <WebKit/WebKit.h>

@protocol WKWebViewProtocol <NSObject>

@property (nonatomic, readonly, copy) NSString *title;
@property (nonatomic, readonly, copy) NSURL *URL;
@property (nonatomic, readonly, getter=isLoading) BOOL loading;
@property (nonatomic, readonly) BOOL canGoBack;
@property (nonatomic, readonly) BOOL canGoForward;

- (void)loadRequest:(NSURLRequest *)request;
- (void)stopLoading;
- (void)evaluateJavaScript:(NSString *)javaScriptString completionHandler:(void (^)(id, NSError *))completionHandler;
- (void)goBack;
- (void)goForward;

@end

```

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

次にUIWebViewを拡張した`UIWebView+ProtocolConfirmed`クラスを作成します。  
基本的にWKWebViewに合わせる形で実装していきます。  

```objective-c
// UIWebView+ProtocolConfirmed.h

#import <UIKit/UIKit.h>
#import "WKWebViewProtocol.h"

@interface UIWebView(UIWebView_ProtocolConfirmed) <WKWebViewProtocol>

@end

```

```objective-c
// UIWebView+ProtocolConfirmed.m

#import "UIWebView+ProtocolConfirmed.h"

@implementation UIWebView(UIWebView_ProtocolConfirmed)

// NSURLの取得
- (NSURL *)URL
{
	NSString *URLString = [self stringByEvaluatingJavaScriptFromString:@"document.URL"];
	return [NSURL URLWithString:URLString];
}

// ページのタイトルの取得
- (NSString *)title
{
	 return [self stringByEvaluatingJavaScriptFromString:@"document.title"];
}

// JavaScriptの実行処理
- (void)evaluateJavaScript:(NSString *)javaScriptString completionHandler:(void (^)(id, NSError *))completionHandler
{
	NSString *result = [self stringByEvaluatingJavaScriptFromString:javaScriptString];
	if(completionHandler) {
		completionHandler(result, nil);
	}
}

```

最後にWKWebView側を実装します。  
と言っても、UIWebViewをWKWebViewに合わせるので、何も書くことはありません。  

```objective-c
// WKWebView+ProtocolConfirmed.h

#import <WebKit/WebKit.h>
#import "WKWebViewProtocol.h"

@interface WKWebView(WKWebView_ProtocolConfirmed) <WKWebViewProtocol>

@end

```

```objective-c
// WKWebView+ProtocolConfirmed.m

#import "WKWebView+ProtocolConfirmed.h"

@implementation WKWebView(WKWebView_ProtocolConfirmed)

@end

```

では、実際に実装していきましょう。  
OAuth認証の処理に関しては、[OAuth2.0認証の処理を自作しよう！](http://grandbig.github.io/blog/2015/06/01/custom-oauth/)を参考にしてください。  

```objective-c
// ViewController.m

#import "ViewController.h"
#import "WKWebViewProtocol.h"
#import "WKWebView+ProtocolConfirmed.h"
#import "UIWebView+ProtocolConfirmed.h"
#import "OAuth2Client.h"

static NSString *const clientId = @"クライアントID";
static NSString *const clientSecret = @"クライアントシークレット";
static NSString *const authorizationURL = @"https://accounts.google.com/o/oauth2/auth";
static NSString *const tokenURL = @"https://accounts.google.com/o/oauth2/token";
static NSString *const scope = @"https://www.googleapis.com/auth/plus.login+https://www.googleapis.com/auth/userinfo.email+https://www.googleapis.com/auth/calendar";

// UIWebViewおよびWKWebView関連のDelegateを読み込む
@interface ViewController ()<UIWebViewDelegate, WKUIDelegate, WKNavigationDelegate>

// UIWebViewとWKWebViewを一纏めにしたWebViewとして下記を定義
@property(nonatomic, assign) id <WKWebViewProtocol> webView;
@property (strong, nonatomic) NSMutableData *receivedData;
@property (assign, nonatomic) BOOL *isLogin;

@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];
	
	// WebViewを画面に追加する処理
	[self createWebView];

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

// WKWebViewまたはUIWebViewをViewに追加する処理
- (void)createWebView {
	
	if(NSClassFromString(@"WKWebView") != nil) {
		// iOS8の場合
		WKWebView *wkWebView = [[WKWebView alloc] initWithFrame:self.view.bounds];
		wkWebView.navigationDelegate = self;
		wkWebView.UIDelegate = self;
		[self.view addSubview:wkWebView];
		self.webView = wkWebView;
	} else {
		// iOS7以下の場合
		UIWebView *uiWebView = [[UIWebView alloc] initWithFrame:self.view.bounds];
		uiWebView.delegate = self;
		[self.view addSubview:uiWebView];
		self.webView = uiWebView;
	}
}

#pragma mark - UIWebViewDelegate

// ページの読込みが開始されたときに呼び出される処理
- (void)webViewDidStartLoad:(UIWebView *)webView {
	NSLog(@"webViewDidStartLoad");
}

// ページを読み終わった後に呼び出される処理
- (void)webViewDidFinishLoad:(UIWebView *)webView {
	NSLog(@"webViewDidFinishLoad");
}

// エラーが発生したときに呼び出される処理
- (void)webView:(UIWebView *)webView didFailLoadWithError:(NSError *)error {
	NSLog(@"didFailLoadWithError");
}

// ページ遷移する前に呼び出される処理
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
	NSLog(@"shouldStartLoadWithRequest");

	BOOL flag = [[OAuth2Client sharedInstance] checkRedirectURI:request];

	if(flag) {
		[[OAuth2Client sharedInstance] getAccessToken:request completionHandler:^(NSString *accessToken){
			// 処理が終了したら呼び出される
			if(accessToken.length > 0) {
				NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
				[defaults setBool:YES forKey:@"isLogin"];
				[defaults synchronize];
			}
			[webView removeFromSuperview];
		}];

		// ページ遷移しない
		return NO;
	}

	// ページ遷移する
	return YES;
}

#pragma mark - WKNavigationDelegate

// ページの読込みが開始されたときに呼び出される処理
- (void)webView:(WKWebView *)webView didStartProvisionalNavigation:(WKNavigation *)navigation
{
	NSLog(@"didStartProvisionalNavigation");
}

// ページを読み終わった後に呼び出される処理
- (void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation
{
	NSLog(@"didFinishNavigation");
}

// エラーが発生したときに呼び出される処理
- (void)webView:(WKWebView *)webView didFailProvisionalNavigation:(WKNavigation *)navigation withError:(NSError *)error
{
	NSLog(@"didFailProvisionalNavigation");
}

// ページ遷移する前に呼び出される処理
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {

	NSLog(@"decidePolicyForNavigationAction");
	NSURLRequest *request = navigationAction.request;

	BOOL flag = [[OAuth2Client sharedInstance] checkRedirectURI:request];

	if(flag) {
		[[OAuth2Client sharedInstance] getAccessToken:request completionHandler:^(NSString *accessToken){
			// 処理が終了したら呼び出される
			if(accessToken.length > 0) {
				NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
				[defaults setBool:YES forKey:@"isLogin"];
				[defaults synchronize];
			}
			[webView removeFromSuperview];
		}];

		// ページ遷移しない
		decisionHandler(WKNavigationActionPolicyCancel);
	}
	
	// ページ遷移する
	decisionHandler(WKNavigationActionPolicyAllow);
}

@end

```

さあ、これでiOS8端末でもiOS7端末でも正しくOAuth認証画面が表示されるかと思います。  
もう今年にはiOS9が発表され、WKWebViewが当然のようになってくるとしたら、UIWebViewは不要になると思いますが、今のところはまだまだ必要でしょう。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
