---
layout: post
title: "UIDynamicAnimatorを使ったピンボールゲームを開発しよう！"
date: 2013-10-30 22:44
comments: true
categories: ios UIDynamicAnimator
---

###ネット上でUIDynamicAnimatorを使ったピンボールゲームが大人気！
先日、iOS7で新たに加わった機能であるUIDynamicAnimatorを使った簡単なアクションゲーム作りについて紹介しました。その後、どんどんこういった物理法則を用いたサンプルがネット上で溢れてくると踏んでいたのですが、まだあまり多くないのが現状です。英語ブログでの紹介はそこそこ増えてきているのですが、如何せん日本語ブログでの扱いは少ないんですよね...。  
そこで[英語でTutorialとして紹介されたページ](http://www.codigator.com/tutorials/uikit-dynamics-ios7-tutorial/)からUIDynamicAnimatorを用いた簡単なゲームを紹介したいと思います。  
今回はピンボールゲームです。メインで扱うものは  
・UICollisionBehavior  
・UIPushBehavior  
の２つです。  
それでは早速説明していきましょう。

<!--more-->

####ソースを書こう！
新規プロジェクトとしてDynamicPractice1を作ります。  
1: 新規作成時に**Empty Application**を選択  
2: ProjectNameをDinamicPractice1とします。  
3: プロジェクトを右クリックしてNew File...からUIViewControllerを追加してください。  
4: AppDelegate.mファイルで追加したUIViewControllerをメインビューとして設定  

```
// AppDelegate.mファイル
#import "ViewController.h"
...
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
	self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
	self.window.backgroundColor = [UIColor whiteColor];
	[self.window makeKeyAndVisible];
	UIViewController *viewCtl = [[ViewController alloc] init];
	self.window.rootViewController = viewCtl;
	
	return YES;
```
さて、上記のように準備ができたら、メインビュー側ソースを書いていきましょう。  
```
// ViewController.hファイル

@interface ViewController : UIViewController <UICollisionBehaviorDelegate> {
	// iPhoneの世界に物理法則を加えるために必要
	IBOutlet UIDynamicAnimator *animator;
	IBOutlet UIPushBehavior *push;
	IBOutlet UICollisionBehavior *collision;

	// ピンボールゲームなのでスライダーバーとボールを用意
	// barもballもUIViewだとつまらないのでballは画像にしてみます
	IBOutlet UIView *bar;
	IBOutlet UIImageView *ball;
}
```
ヘッダーファイルに必要なものの定義が終わったので、続いてViewController.mファイルを書いていきます。

```
// ViewController.mファイル
- (void)viewDidLoad {
	[super viewDidLoad];
	// 物理法則を設定
	animator = [[UIDynamicAnimator alloc] initWithReferenceView:self.view];

	// ボールに画像を設定
	ball = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"sun.png"]];
	[self.view addSubview:ball];

	// スライダーバーを設定
	bar = [[UIView alloc] initWithFrame:CGRectMake(6, 534, 145, 12)];
	[bar setBackgroundColor:[UIColor blackColor]];
	[self.view addSubview:bar];

	// 初めにボールが下に落ちていくように設定
	push = [[UIPushBehavior alloc] initWithItems:@[ball] mode:UIPushBehaviorModeInstantaneous];
	[push setPushDirection:CGVectorMake(-0.2, 0.2)];

	// ボールが衝突を検知するように設定
	collision = [[UICollisionBehavior alloc] initWithItems:@[ball]];
	// スライダーバーを衝突の境界線として設定
	[collision addBoundaryWithIdentifier:@"bar" fromPoint:bar.frame.origin toPoint:CGPointMake(bar.frame.origin.x+bar.frame.size.width, bar.frame.origin.y)];
	// 天井を衝突の境界線として設定
	[collision addBoundaryWithIdentifier:@"top" fromPoint:CGPointMake(0, 0) toPoint:CGPointMake(self.view.frame.size.width ,0 )];
	// 左壁一帯を衝突の境界線として設定
	[collision addBoundaryWithIdentifier:@"left" fromPoint:CGPointZero toPoint:CGPointMake(0, self.view.frame.size.height)];
	// 右壁一帯を衝突の境界線として設定
	[collision addBoundaryWithIdentifier:@"right" fromPoint:CGPointMake(self.view.frame.size.width, 0) toPoint:CGPointMake(self.view.frame.size.width, self.view.frame.size.height)];

	collision.collisionDelegate = self;

	[animator addBehavior:push];
	[animator addBehavior:collision];
}

// 衝突したときの処理を書きます
- (void)collisionBehavior:(UICollisionBehavior*)behavior beganContactForItem:(id <UIDynamicItem>)item withBoundaryIdentifier:(id <NSCopying>)identifier atPoint:(CGPoint)p {
	// スライダーバーとボールが衝突したとき
	if ([(NSString*)identifier isEqualToString:@"bar"]) {
		[self->push setPushDirection:CGVectorMake(0.2, -0.2)];
		[self->push setActive:YES];
	} else if([(NSString*)identifier isEqualToString:@"top"]) {
		// 天井とボールが衝突したとき
		[self->push setPushDirection:CGVectorMake(-0.8, 0.6)];
		[self->push setActive:YES];
	} else if([(NSString*)identifier isEqualToString:@"left"]) {
		// 左壁とボールが衝突したとき
		[self->push setPushDirection:CGVectorMake(0.4, 0.4)];
		[self->push setActive:YES];
	} else {
		// 右壁とボールが衝突したとき
		[self->push setPushDirection:CGVectorMake(-0.4, -0.4)];
		[self->push setActive:YES];
	}
}

// 以下はスライダーバーを動かす処理です
-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
	[self->collision removeBoundaryWithIdentifier:@"bar"];
	UITouch *touch = [touches anyObject];
	CGPoint touchPoint = [touch locationInView:self.view];
	CGRect frame = bar.frame;
	[bar setCenter:CGPointMake(touchPoint.x, frame.origin.y + frame.size.height/2)];
	[self->collision addBoundaryWithIdentifier:@"bar" fromPoint:bar.frame.origin toPoint:CGPointMake(bar.frame.origin.x+bar.frame.size.width, bar.frame.origin.y)];
}
```
さて、ピンボールゲームの完成です。  
実際、UIPushBehaviorではなく、UIGravityBehaviorでも作れると思いますが、**Gravity**ってところから**重力**ってイメージだからUIPushBehaviorの方が自然なのかな〜。と思ったり思わなかったり...。  

筆者はボールをどんどん増やしてゲームの難易度をあげようかと思いましたが、これが難しい。  
物理法則をアプリの世界に作って、その物理法則を受ける世界に入れてあげるアイテムとしてballを追加しているというイメージなんですよね...。  

iOS7で対応する以前から物理法則をアプリに再現しているゲームは多々あったと思います。その中でこのクラスがどういった役割を果たしていくのか個人的に気になるところです。

