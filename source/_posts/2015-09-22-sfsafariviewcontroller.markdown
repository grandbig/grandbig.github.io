---
layout: post
title: "SFSafariViewControllerを使ってみよう！"
date: 2015-09-22 22:13
comments: true
categories: ios9 webview
---

####iOS9から導入されたSFSafariViewControllerを使ってみよう！
さて、iPhone6s, iPhone6s Plusの発売を心待ちにされている開発者の皆さんを横目にブログを更新したいと思います。  
と言っても、筆者は土曜日にはiPhone6sをGETできるはず...。  

本日はSFSafariViewControllerを触ってみたいと思います。  
SFSafariViewControllerはアプリ内でWeb画面を表示したいときに使うことをAppleが奨励しています。これにより、今まで自作アプリからSafariを起動させていたのに対して、アプリ内でSafariを呼び出すことが可能となりました。  
気をつけて欲しいのは、 **カスタマイズはできない** ということです。  
例えば、UIWebViewやWKWebViewでやっていたような『Web側から **document.location = ....** を実行することでネイティブ側で何か処理をさせる』といったことはできません。  
あくまでも **単なるWebサイトの表示** に利用します。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

#####SFSafariViewControllerの使い方
まずは、使い方を説明しましょう。  
SFSafariViewControllerを使うためには、 **SafariServices.framework** を追加する必要があります。  

![SafariServices.frameworkを追加](/images/sfsafariviewcontroller_1.png)  

後は、ソースコードを書くだけです。  
今回は画面上に置かれた『WEB』ボタンをクリックしたときにSFSafariViewControllerを呼び出すようにしてみます。  

![画面キャプチャ](/images/sfsafariviewcontroller_2.png)  

実際のソースコードは下記です。  

```objective-c
// ViewController.m
#import "ViewController.h"
#import <SafariServices/SafariServices.h>

@interface ViewController () <SFSafariViewControllerDelegate>

@property (strong, nonatomic) SFSafariViewController *sfv;

@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];
}

- (void)didReceiveMemoryWarning {
	[super didReceiveMemoryWarning];
}

// WEBボタンをタップした際に呼び出される処理
- (IBAction)goToWebPage:(id)sender {
	self.sfv = [[SFSafariViewController alloc] initWithURL:[NSURL URLWithString:@"http://www.yahoo.co.jp/"]];
	self.sfv.modalTransitionStyle = UIModalTransitionStyleCrossDissolve;
	[self presentViewController:self.sfv animated:YES completion:nil];
	self.sfv.delegate = self;
}

#pragma mark - SFSafariViewControllerDelegate
// アクションボタンをタップされた際に呼び出される処理
- (NSArray<UIActivity *> *)safariViewController:(SFSafariViewController *)controller activityItemsForURL:(NSURL *)URL title:(NSString *)title {

	NSLog(@"activityItemdForURL");		    
	return @[];
}

// SFSafariViewControllerに表示する最初の画面の読込みが完了した際に呼び出される処理
- (void)safariViewController:(SFSafariViewController *)controller didCompleteInitialLoad:(BOOL)didLoadSuccessfully {
	    
	NSLog(@"didCompleteInitialLoad");
}

// SFSafariViewControllerのDone(完了)ボタンをタップした際に呼び出される処理
- (void)safariViewControllerDidFinish:(SFSafariViewController *)controller {
	    
	NSLog(@"safariViewControllerDidFinish");
}

@end

```

これでSFsafariViewControllerが実装されたことがわかると思います。  

因みに、SFSafariViewControllerを使う上で気をつけたいのが、如何にユーザにあくまでもアプリ内の遷移であると理解させることかと思います。  
例えば、デフォルトではSFSafariViewControllerは下から上に上がってくる形で表示がされます。  
もし、WebView内でふわっと浮かび上がってくるような画面遷移のアニメーションをしているのだとしたら、これはかなりの違和感になりますよね？  
なので、`self.sfv.modalTransitionStyle = UIModalTransitionStyleCrossDissolve;`のようにアニメーションを工夫することも重要になってきます。  

また、`SFSafariViewControllerDelegate`のメソッドを3つ書いてみましたが、あまり使いどころが思い浮かばないというのが正直なところです。  
Done(完了)ボタンを押したときに`dismissViewController`が必要かとも思ったのですが、自動的に閉じますし...。  

`SFSafariViewController`をユーザに見せている間に何らかの処理を裏でしておくといったことは考えられるかもしれませんね。  

#####Safariを起動する
SFSafariViewControllerが登場したからといって、Safari起動ができなくなったわけではもちろんありません。  
Safariを起動したって良いんです。  
というのもiOS9からはアプリ間遷移がスムーズになったからです。  
ただし、 **1つ前のアプリに戻る** ときのみの話ですが、頻繁に多種多様なアプリ間を移動するアプリでなければ十分でしょう。  

試しに、先ほどの『WEB』ボタンをタップしたときの処理をSafariの起動に変更してみます。  

```objective-c
// ViwController.mから一部抜粋
// WEBボタンをタップした際に呼び出される処理
- (IBAction)goToWebPage:(id)sender {
	// Safariを起動
	[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"http://www.yahoo.co.jp/"]];
}
```

Safariアプリを起動した後、下記のように、左上に戻るボタンが表示されます。  

![1つ前のアプリに戻る](/images/sfsafariviewcontroller_3.png)  


さて、いかがだったでしょうか。  
今後は、iOS9以降対応のアプリを開発すると、すぐには完全に割り切ることはできないと思いますので、iOS8までのことも踏まえつつ、最適な方法を取るのが良いでしょう。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

