---
layout: post
title: "Xcode6 Storyboardで多言語対応してみた"
date: 2014-12-07 17:50
comments: true
categories: ios xcode6 多言語
---

####Xcode6を使った多言語対応
本日はXcode6(正確にはXcode6.1.1)で多言語対応する方法を書きます。  
全スクリーンショットを載せます。  

まず、多言語化に使うサンプルプロジェクトは下記です。  
StoryboardにはLabelとButtonを1つずつ配置します。  
![StoryboardにLabelとButtonを配置](/images/multiLanguage1.png)  

<!-- more -->

ソースは下記とします。  

```objective-c
// ViewController.m

#import "ViewController.h"

@interface ViewController () <UIAlertViewDelegate>

@property(strong, nonatomic) UIAlertView *alert;

@end

@implementation ViewController

- (void)viewDidLoad {
	[super viewDidLoad];

	self.alert.delegate = self;
}

- (void)didReceiveMemoryWarning {
	[super didReceiveMemoryWarning];
}

// アラートの表示
- (IBAction)showAlert:(id)sender {
	
    self.alert = [[UIAlertView alloc] initWithTitle:@"確認"
                                            message:@"テスト"
                                           delegate:self
                                  cancelButtonTitle:nil
                                  otherButtonTitles:@"OK", nil];

	[self.alert show];
}
@end

```

では、早速、多言語化を進めていきましょう。  

1. PROJECT > info > Localizationsに項目を追加  
初めはEnglishのみ項目があります。『＋』をクリックして、追加したい言語を選択しましょう。  
![Localizationsに項目追加](/images/multiLanguage2.png)  
![追加したい言語を選択](/images/multiLanguage3.png)  
![チェックがついていることを確認](/images/multiLanguage4.png)  

2. Main.storyboardの追加
次にstoryboardに言語の設定を追加しましょう。  
Main.storyboardを選択し、右メニューの『Show the File inspector』> Localizationを見てください。  
ここで使いたい言語にチェックをつけましょう。  
![Main.storyboardに言語設定を追加](/images/multiLanguage5.png)  

3. LaunchScreenの追加  
そして、Main.storyboardで実施したのと同様にLaunchScreenでも言語を設定しましょう。  
LaunchScreen.xibを選択し、右メニューの『Show the File inspector』> Localizationで使いたい言語にチェックをつけましょう。  
![LaunchScreen.xibに言語設定を追加](/images/multiLanguage6.png)  

4. Info.plistの追加
同様にInfo.plistでも言語を設定しましょう。  
Info.plistを選択し、右メニューの『Show the File inspector』> Localizationで使いたい言語にチェックをつけましょう。  
![Info.plistに言語設定を追加](/images/multiLanguage13.png)  
アプリ名など言語ごとに変更したいKeyの値を変えましょう。  

5. Localizable.stringsファイルの追加  
続いて、ソース内の文言を多言語化する手順です。  
Supporting FilesにStrings FileをLocalizable.stringsというファイル名で追加しましょう。  
今は保存場所はあまり気にしなくて構いません。  
ファイルを追加したら、Localizable.stringsを選択し、右メニューの『Show the File inspector』> Localizationで使いたい言語にチェックをつけましょう。  
![Strings Fileを追加](/images/multiLanguage7.png)  
![Baseを選択したまま進む](/images/multiLanguage8.png)  
![Localizable.stringsに言語設定を追加](/images/multiLanguage9.png)  

6. Localizable.stringsファイルに文言を追加
先ほど追加したファイルに文言を追加しましょう。

```objective-c
// Localizable.strings(Base)ファイル
"alertTitle"="Confirm";
"alertMessage"="Test";
"alertOKBtn"="OK";

// Localizable.strings(English)ファイル
"alertTitle"="Confirm";
"alertMessage"="Test";
"alertOKBtn"="OK";

// Localizable.strings(Japanese)ファイル
"alertTitle"="確認";
"alertMessage"="テスト";
"alertOKBtn"="OK";

```

反映させたいファイルへは下記のように書きます。

```objective-c
// ViewController.mファイル

// アラートの表示
- (IBAction)showAlert:(id)sender {
	
	NSString *alertTitle = NSLocalizedString(@"alertTitle", nil);
	NSString *alertMessage = NSLocalizedString(@"alertMessage", nil);
	NSString *alertOKBtn = NSLocalizedString(@"alertOKBtn", nil);

    self.alert = [[UIAlertView alloc] initWithTitle:alertTitle
                                            message:alertMessage
                                           delegate:self
                                  cancelButtonTitle:nil
                                  otherButtonTitles:alertOKBtn, nil];

	[self.alert show];
}

```

ここまでできれば、多言語化OKです。  

####注意点
途中で躓いた場合の対応を書いておきます。  

1. Info.plistが見つからないというエラーが出る
下記のようなエラーが出た場合の対処を書いておきます。  
![Info.plistが見つからないエラー](/images/multiLanguage10.png)  
これは、多言語化したことにより、当初はAppDelegate.mファイルと同じ階層にあったInfo.plistファイルが多言語ごとのフォルダ配下に移動したために発生しています。  
なので、TARGETS > Build Settings > Packaging > Info.plist Fileのパスを変更しましょう。  
![パスの変更](/images/multiLanguage11.png)  

2. 途中でStoryboardにラベルを追加した場合の対応
開発していれば、途中で表示したいものを増やすこともあるでしょう。  
その場合は、Main.storyboardを選択して、右メニューの『Show the File inspector』> LocalizationからBase以外のチェックを外しましょう。  
それから、ラベルを追加して、再度、Base以外のチェックをつけましょう。  
その際、前のファイルを使うか、新しいファイルに置き換えて使うかを問われます。  
新しいファイルに置き換えて使う方の **Replace file** を選択しましょう。  
![Replace fileを選択](/images/multiLanguage12.png)  

以上で多言語化の注意点は終了です。  
世界に羽ばたくアプリを開発するためには多言語化は必須ですね。  
面倒な対応ではありますが、ぜひ取り組んでみてください。  

ということで、今日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

