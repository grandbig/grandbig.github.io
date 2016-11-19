---
layout: post
title: "iOS7の新機能『UIDynamicAnimator』を使って簡単なアクションゲームを開発しよう！"
date: 2013-10-19 16:23
comments: true
categories: ios UIDynamicAnimator
---

###UIDynamicAnimator, UIGravityBehavior, UICollisionBehaviorなど新機能がiOS7に！！
今回はiOS7で新しく加わった新機能を使って簡単なアクションゲームを開発してみたいと思います。iOS7では物理法則に基づいたアニメーションを実装できるように**UIDynamicAnimator, UIGravityBehavior, UICollisionBehavior, UIPushBehavior, UIAttachmentBehavior, UISnapBehavior, UIDynamicItemBehavior**が追加されました。  
前々からアクションゲームを開発してみたいと思っていた筆者はこれを使わない手はないということで早速使ってみることにしました。

<!--more-->

####基本説明
・UIDynamicAnimator  
アプリに物理アニメーションを実装するためのコアとなるクラスです。アプリのViewに物理現象を発生させられる世界を設定するようなイメージですかね...。  
・UIGravityBehavior  
重力現象を設定したいときに使います。重力の強さや方向を設定することができます。  
・UICollisionBehavior  
物同士の衝突現象を設定したいときに使います。  
・UIPuchBehavior  
名前の通り押したような現象を設定したいときに使います。瞬間的に力を加えたり、継続的に力を加えたりできます。  
・UIAttachmentBehavior  
付着、接続などの現象を設定したいときに使います。  
・UISnapBehavior  
指定した点に吸い付けられるような設定をしたいときに使います。  
・UIDynamicItemBehavior  
指定したものの物理的特性を設定するために使います。  

####開発
では、早速、開発してみましょう。DynamicPractice.xcodeprojを作成します。いつも通りの手順ですが、以下におさらい。  
1: 「Empty Application」を選択してDynamicPracticeという名前をつけたプロジェクトを生成  
2: UIViewControllerを追加  
3: AppDelegate.mファイルでrootViewControllerに先ほど追加したUIViewControllerを設定  

さて、物理現象の実装に移りましょう。  

```
// ViewController.hファイル
@interface ViewController : UIViewController {
	// ①
	IBOutlet UIDynamicAnimator *animator;
	IBOutlet UIGravityBehavior *gravity;
	IBOutlet UICollisionBehavior *collision;
	IBOutlet UIPushBehavior *push;
	// ②
	IBOutlet UIImageView *img;
	IBOutlet UIButton *jButton, *rButton, *lButton;
}

// ③
- (IBAction)JumpAction:(id)sender;
- (IBAction)RightMoveAction:(id)sender;
- (IBAction)LeftMoveAction:(id)sender;

```

① 今回利用する物理現象を定義  
② アクションをさせる画像を設定, ジャンプと左右移動をするためのボタンの定義, 移動/停止の判断フラグを設定  
③ ジャンプ, 左右移動時の処理をする関数を定義  

続いて上記の定義をDynamicPractice.xibに置く各ボタンと関係させます。(といってもボタンだけですが…)  
![簡単にこんな画面を用意](/images/xcode_dynamic1.png)  

最後に具体的な各処理の実装をします。  
```
// ViewController.mファイル
- (void)viewDidLoad {
	[super viewDidLoad];

	// ①
	animator = [UIDynamicAnimator alloc] initWithReferenceView:self.view];]
	
	img = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"***********.png"]];
	img.center = CGPointMake(100.0, 100.0);
	[self.view addSubview:img];

	// ②
	collision = [UICollisionBehavior alloc] initWithItems:@[img]];
	collision.translatesReferenceBoundsIntoBoundary = YES;

	// ③
	gravity = [[UIGravityBehavior alloc] initWithItems:@[img]];
	gravity.magnitude = 1.0f;
	gravity.gravityDirection = CGVectorMake(0.0f, 1.0f);

	// ④
	push = [[UIPushBehavior alloc] initWithItems:nil mode:UIPushBehaviorModeContinuous];

	// ⑤
	[animator addBehavior:gravity];
	[animator addBehavior:collision];
	[animator addBehavior:push];
}
```
① self.viewに物理現象の世界を加えられるように設定  
② 衝突現象をimgに設定。translatesReferenceBoundsIntoBoundaryでNOにするとimgがそのまま落ちていって悲しいことになります。  
③ 重力現象をimgに設定。強さと方向を設定します。初めは上からimgが降ってくる想定なので、X方向は0.0fでY方向に1.0fと設定します。  
④ 横移動で使うためにインスタンス化だけ先にしておきます。まだ使いたくないのでinitWithItemsにはnilを設定しています。  
⑤ gravity, collision, pushをanimatorに追加します。  

これでViewController読み込まれたら初期物理現象が設定されます。続いて各ボタンの処理を書きます。  
```
// ViewController.mファイル
// ①
- (IBAction)JumpAction:(id)sender {
	gravity.magnitude = 1.0f;
	gravity.gravityDirection = CGVectorMake(0.0f, -1.0f);
	[gravity addItem:img];

	[self performSelector:@selector(changeGravity) withObject:nil afterDelay:0.2];
}

// ②
- (void)changeGravity {
	gravity.magnitude = 1.0f;
	gravity.gravityDirection = CGVectorMake(0.0f, 1.0f);
	[gravity addItem:img];
}

// ③
- (IBAction)RightMoveAction:(id)sender {
	push.magnitude = 0.05f;
	push.pushDirection = CGVectorMake(1.0f, 0.0f);
	[push addItem:img];

	[self performSelector:@selector(stopPush) withObject:nil afterDelay:0.2];
}

// ④
- (IBAction)LeftMoveAction:(id)sender {
	push.magnitude = 0.05f;
	push.pushDirection = CGVectorMake(-1.0f, 0.0f);
	[push addItem:img];

	[self performSelector:@selector(stopPush) withObject:nil afterDelay:0.2];
}

// ⑤
- (void)stopPush {
	[push removeItem:img];
}
```
① これはJumpボタンをクリックしたときの処理です。重力を上向きに加えます。ただし、このまま放置すると天井まで登り続けるので、筆者は0.2秒後に通常重力をかけるようにしました。  
② ①実行後の0.2秒後に通常重量(下向きに重力)をかけるようにする処理です。  
③ 右移動の処理です。右向きに力を加えるために、インスタンス化だけしておいたpushに強さと方向を設定します。これも①同様、ずっと力をかけ続けると右端から動けなくなるので0.2秒後にストップ処理を実行するようにしました。  
④ ③と同じく左移動の処理です。  
⑤ ストップさせるためにpushの力を加えるItemからimgを外しています。  

以上で、ジャンプ、左右移動が可能な簡単なアクションゲームを開発することができます。  

因みに筆者がテストで作った画面はこんな感じです。  
![簡易アクションゲーム](/images/xcode_dynamic2.png)  

さてさて、せっかくなのでうまくこういった機能を使ってもっと面白いものを開発したいものです。  

参考: 
[iOS7 UI Dynamics](http://dev.classmethod.jp/references/ios7-uidynamic-2/)  
[魅せるUIの作り方](http://www.slideshare.net/techblogyahoo/5-ios7-workshopmakeupui)  

