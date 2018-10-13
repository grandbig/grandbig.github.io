---
layout: post
title: "Xcode5でAuto Layoutを使おう"
date: 2014-02-09 17:25
comments: true
categories: xcode autolayout
---

###Auto Layoutで『iOS6・iOS7』と『3.5-inch・4-inch』対応
さて、これまで**iOS6/7 deltas**を利用した方法や**全てコードで書く**方法について説明してきました。これとは別に本日は**Auto Layout**を利用した方法について説明します。  
先にこれらの方法の違いについて紹介します。  
・iOS6/7 deltasを利用する方法  
  XIBやStoryboardを利用した上でiOS5以前から対応する場合  
・全てコードで書く方法  
  XIBやStoryboardなどのGUIを利用せずに、全OS・全ディスプレイに対応する場合  
・Auto Layoutを利用する方法  
  XIBやStoryboardを利用した上でiOS6以降から対応する場合  

求められている仕様や開発者の技量、プロジェクトの運用から最適な方法を選択する必要があります。  
とは言え、個人的には**全てコードで書く**方法が最も自由がきくと思っているので、これに慣れていると他の手法は案外シンドイです。  

<!--more-->
####Auto LayoutでUINavigationBarを調整しよう
以前、iOS6/7 deltasを利用する方法で説明したことをAuto Layoutで説明します。皆様、ご存知の通りiOS6以前とiOS7以降とではステータスバーの20px分をナビゲーションバーの高さを考慮しなくてはなりません。  
![iOS6とiOS7でナビゲーションバーの高さが異なる](/images/ios6_7_difference.png)  

早速、詳しく説明していきます。  
まず、プロジェクトを作成し終わったら、XIBファイルを表示してください。そしてUINavigationBarを設置します。このUINavigationBarの高さをiOSのバージョンごとに変化させたいので、下記手順でAuto LayoutのConstraintを追加します。  
1: 右下のバーの左から2番目をクリックしてConstraintsのウィンドウを表示  
2: 高さを変更したいのでHeightにチェックを入れる  
3: Add ConstraintボタンをクリックしてConstraintを追加する  
![Constraintを追加する](/images/autolayout1.png)  

これでConstraintsが追加されたのでViewController.hと関連づける。  
1: Constraints > Heightを右クリックしてViewController.hへドラッグ&ドロップ  
2: ウィンドウが表示されるので、名前をつけてConnectをクリックする  
![ViewController.hに関連づける](/images/autolayout2.png)  
![名前をつけてConnect](/images/autolayout3.png)  

次にViewController.mファイルを開きましょう。  
以下のように編集してください。  
```
- (void)viewDidLoad
{
	[super viewDidLoad];
	// iOS7の場合は64pxにする
	if (floor(NSFoundationVersionNumber) > NSFoundationVersionNumber_iOS_6_1) {
		self.nvBarHeight.constant = 64;
	}
}
```
以上の手順をすることで下記のようにiOS6とiOS7でUINavigationBarを正しく表示することができます。  
![UINavigationBarの表示](/images/autolayout4.png)  

####Auto LayoutでUIViewを中央に配置しよう
次に主にディスプレイサイズで困るであろう中央配置の方法について説明します。  
まずは、何も考えずに普通にUIViewを追加してみましょう。そしてViewを3.5-inch Full Screenと4-inch Full Screenとで表示を変更して見てみましょう。すると、下記のようにUIViewの高さが変わってしまうことがわかると思います。  
![4-inchの場合](/images/autolayout5.png)  
![3.5-inchの場合](/images/autolayout6.png)  
こんなはずでは...なんて絶句してしまうかもしれません。が、ご安心ください。これは先ほど説明したHeightのConstraintsを追加することで対応できます。先ほどと同様に下記の手順でHeightのConstraintを追加してみてください。  

1: 追加したUIViewを選択した状態で右下のバーの左から2番目をクリックしてConstraintのウィンドウを表示  
2: 高さを保持したいのでHeightにチェックを入れます。高さが希望の値になっていることを確認してください。  
3: Add ConstraintボタンをクリックしてConstraintsを追加する  

これで先ほどと同様にViewのinchを変更させてみてください。追加したUIViewの高さが保持されていることがわかると思います。  

はい。本題です。  
UIViewをディスプレイのCenterに表示させたい場合は下記手順を実行してください。  

1: 追加したUIViewを選択した状態で右下のバーの左から1番目をクリックしてConstraintのウィンドウを表示  
2: Vertical方向に常にディスプレイの中央に表示させたいので、Vertical Center in Containerにチェックを入れます。  
3: Add ConstraintボタンをクリックしてConstraintsを追加する  
![Vertical Center in Containerを追加](/images/autolayout7.png)  

これでUIViewを高さを保持した状態で3.5-inch/4-inchでもディスプレイの中央に表示することができます。

####Auto Layoutでマージンを調整しよう
そして、Auto Layoutを用いたマージンの調整方法についても説明しましょう。  
先ほど、ディスプレイの中央に表示したい場合を説明したと思いますが、3.5-inchで見るとUINavigationBarに近すぎると感じることがあるかもしれません。  
であるならば、3.5-inchでも4-inchでもUINavigationBarから一定の距離を保持した方が見栄えが良いと思うこともあると思います。  
さて、その方法について説明します。  

1: 追加したUIViewを選択した状態で右下のバーの左から2番目をクリックしてConstraintのウィンドウを表示  
2: 上部との距離を変更します。(初期状態は点線表記になっていますが、赤の実線になるように値を変更してください)  
3: Add ConstraintボタンをクリックしてConstraintsを追加する  
![上部マージンを追加](/images/autolayout8.png)  

これで3.5-inch/4-inchにディスプレイが変更されたとしてもUINavigationBarとUIViewとの距離は一定に保たれます。  


####最後に
いかがだったでしょうか？  
筆者的には完全に手探りな状態で始めたので、わからないことだらけだったのですが、なんとなく理解できてきました。  
現在はiPhone4S以前を利用しているユーザは(日本では？)少なくなってきたと思うのですが、iPadがiPhoneの3.5-inchと同じ比率ですので、Universalアプリを開発するのであれば対応せざるを得ません。  
また、iOS6ユーザはまだまだいると思いますので、仕様として要求される可能性は十分あります。細かい修正が常に要求されるとは思いますが、忍耐強くiOS8, 9, 10...とリリースされるのを待ちましょう笑  
次回リリースされるiPhoneやiPadまたはリリースされるかもしれない新デバイスがどんなディスプレイの比率になるかもわからないので、今後こういった対応は常に要求されるんでしょうね(汗)  

ということでまた次回！  
筆者も継続して勉強します。  
