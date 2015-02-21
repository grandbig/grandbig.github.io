---
layout: post
title: "iOS8からのLocalNotification"
date: 2014-11-09 17:05
comments: true
categories: ios ios8 Push
---

####LocalNotificationでAction
今日はiOS8から追加されたLocalNotificationの新機能についてお話します。  
これまでのLocalNotificationは  

* 通知が通知エリアに表示される  
* 通知をタップすることでアプリを起動  

くらいの機能しかありませんでした。  

しかし、iOS8からは届いた通知に対して、ユーザが答えることができるようになりました。  

<!-- more -->

例えば、下記のような感じですね。  

![バナー表示](/images/localnotification_1.png)  
![ダイアログ表示](/images/localnotification_2.png)  

これまでLocalNotificationを使ってきた人であれば、実に簡単に実装ができます。  
それでは、実際にソースを見て行きましょう。  

```objective-c
// AppDelegate.m
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

	if ([UIApplication instancesRespondToSelector:@selector(registerUserNotificationSettings:)]) {
		[self registerUserNotificationSettings];
	}

	return YES;

}

<省略>

// アプリがBackground起動になったときに処理
- (void)applicationDidEnterBackground:(UIApplication *)application {
	// LocalNotificationの送信
	[self sendLocalNotificationForMessage:@"TEST"];
}

<省略>

// LocalNotificationでアクションを実行したときの処理
- (void) application:(UIApplication *)application handleActionWithIdentifier:(NSString *)identifier forLocalNotification:(UILocalNotification *)notification completionHandler:(void (^)())completionHandler
{
	if ([identifier isEqualToString:@"ACCEPT"]) {
		NSLog(@"Accepted");
	} else if([identifier isEqualToString:@"MAYBE"]) {
		NSLog(@"Maybe");
	} else if([identifier isEqualToString:@"DECLINE"]) {
		NSLog(@"Declined");
	}

	if(completionHandler) {
		completionHandler();
	}
}

// Notificationの設定
- (void)registerUserNotificationSettings
{
	// Actionの生成
	UIMutableUserNotificationAction *acceptAction = [[UIMutableUserNotificationAction alloc] init];
	acceptAction.identifier = @"ACCEPT";
	acceptAction.title = @"Accept";
	acceptAction.activationMode = UIUserNotificationActivationModeBackground;
	acceptAction.authenticationRequired = NO;
	acceptAction.destructive = NO;

	// Actionの生成
	UIMutableUserNotificationAction *acceptAction = [[UIMutableUserNotificationAction alloc] init];
	acceptAction.identifier = @"MAYBE";
	acceptAction.title = @"Maybe";
	acceptAction.activationMode = UIUserNotificationActivationModeBackground;
	acceptAction.authenticationRequired = NO;
	acceptAction.destructive = NO;

	// Actionの生成
	UIMutableUserNotificationAction *acceptAction = [[UIMutableUserNotificationAction alloc] init];
	acceptAction.identifier = @"DECLINE";
	acceptAction.title = @"Decline";
	acceptAction.activationMode = UIUserNotificationActivationModeBackground;
	acceptAction.authenticationRequired = NO;
	acceptAction.destructive = YES;

	// Categoryの作成
	UIMutableUserNotificationCategory *inviteCategory = [UIMutableUserNotificationCategory alloc] init];
	inviteCategory.identifier = @"INVITE_CATEGORY"; // CategoryのIDを設定
	[inviteCategory setActions:@[acceptAction, maybeAction, declineAction] forContext:UIUserNotificationActionContextDefault]; // ダイアログ表示
	[inviteCategory setActions:@[acceptAction, declineAction] forContext:UIUserNotificationActionContextMinimal]; // バナー表示

	NSSet *categories = [NSSet setWithObjects:inviteCategory, nil];
	UIUserNotificationSettings *notificationSettings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeAlert|UIUserNotificationTypeBadge|UIUserNotificationTypeSound categories:categories];
	[[UIApplication sharedApplication] registerUserNotificationSettings:notificationSettings];
}

// LocalNotificationを送信
- (void)sendLocalNotificationForMessage:(NSString *)message
{
	UILocalNotification *localNotification = [UILocalNotification new];
	localNotification.alertBody = message;
	localNotification.fireDate = [NSDate date];
	localNotification.timeZone = [NSTimeZone localTimeZone];
	localNotification.soundName = UILocalNotificationDefaultSoundName;
	localNotification.category = @"INVITE_CATEGORY"; // Action表示させたいCategoryの設定
	[UIApplication sharedApplication] scheduleLocalNotification:localNotification];
}
```

簡単に書けますね。  
ぱっと見た感じは、予めAction項目を作成しておく必要があるので、用途は限定的になりそうな気がしますね。  
と言っても、これまでよりユーザ操作のハンドリングの幅が広がるので、嬉しい限りですが...。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

