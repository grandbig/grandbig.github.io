---
layout: post
title: "UIButtonのaddTargetについて"
date: 2014-02-01 22:04
comments: true
categories: ios
---

###UIButtonをタップしたときの処理を書くときの注意点
今日は簡単なメモを書きます。ネイティブアプリを開発するときに、レイアウトをこりたいと思うことがあります。そんなときにひっかかってしまったことを書きます。(まさか今更こんなことにひっかかるとは...と思いました笑)  

UIButtonは下記のように作成します。  
```
UIButton *btn = [UIButton buttonWithType: UIButtonTypeRoudedRect];
btn.frame = CGRectMake(0, 0, 100, 30);
[btn setTitle:@"ボタン" forState:UIControlStateNormal];
[btn addTarget:self action:@selector(test:) forControlEvents:UIControlEventTouchDown];
```
<!--more-->

で、このボタンをUILabelに配置するために  
```
UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(0, 0, 300, 50)];
[label addSubview: btn];
```
と書いたところ、addTargetで設定したタップイベントが実行されませんでした。  
これを  
```
 UIView *uv = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 300, 50)];
[uv addSubview: btn];
```
と**UIView**に変えたところ、うまくいきました。  

ま、UILabelをあえて使う必要もないので、UIViewで良いでしょう。
