---
layout: post
title: "Xcode8を始めてみよう！"
date: 2016-09-19 14:31
comments: true
categories: xcode8
---

###Xcode8が登場！
先日、Xcode8がiOS10と共に一般公開されました。本ブログの初期の頃に[Xcodeを始めてみよう！](http://grandbig.github.io/blog/2013/09/05/start-xcode/)という記事を書きましたが、この頃がXcode4.6.3であったことを考えると何だか感慨深いですね。  
なんて感傷に浸りたい気持ちとは裏腹にXcode8ではこれまでと大きく変わっている部分があるとのことで、少しではありますが見ていこうと思います。  

<!-- more -->

###プロジェクトの新規作成
まずはプロジェクトの新規作成フローから見ていきます。  

１．Xcode8を起動する  

![Xcode8起動時のウィンドウ](/images/xcode8_1.png)  

２．作成するプロジェクトの形式を選択する  

![プロジェクトの形式を作成](/images/xcode8_2.png)  

３．プロジェクトの詳細を決める  

![プロジェクトの詳細を入力](/images/xcode8_3.png)  

４．プロジェクトを保存する  

![プロジェクトを保存](/images/xcode8_4.png)  

レイアウト構成やTeamの入力項目が増えただけで大きな違いはなさそうです。  

###Xcode8のGeneral
続いて、Xcode8で変わったと話題の **General** を見ていきましょう。  

![General](/images/xcode8_5.png)  

上図を見ると、  

* Display Nameの追加  
* Automatically manage signing機能の新規追加  
* Provisioning ProfileとCertificate状況の閲覧項目の追加  

が増えたことがわかります。  

####Automatically manage signing
これは一見、自動で最適なProvisioning ProfileとCertificateを選択してくれる機能に見えますが、下記機能を含んでいます。  

* 署名付き証明書の作成  
  * 複数Macで開発する際に必要な鍵付き証明書を作成してくれます  
  * 作成可能な証明書数は1 Apple IDあたり10個とのこと  
* App IDの作成と更新  
  * アプリ固有のIDであり、存在しないBundle identifierを指定していたとしても自動作成  
* Provisioning Profileの作成と更新  
  * アプリのビルドに必要な証明書でApp IDに紐づく証明書がないときは自動作成  

実際に、機能をONにした場合、下図のように自動で最適なものが選択されていることがわかります。  

![Automatically manage signingをON](/images/xcode8_6.png)  

因みにこれをOFFにした場合は下図のように全て自身で選択できる項目が表示されます。  

![Automatically manage signingをOFF](/images/xcode8_7.png)  

気をつけたいのは想定通りのTeamが選択されていないと意図したProvisioning Profileを選択できない場合があることです。  
しかし、GeneralからはTeamの選択ができません。  
筆者は結局、 **Build Settings** からTeamを選択しました。(他にも方法はあるかもしれません。)  
ま、ここまで来たら、いっそのことProvisioning Profileも選択しちゃいましたが...笑  
Apple的にはGeneralで全て完結させることを推奨しているようですが、課題がありそうな気がしています。  

![Build SettingsからTeamを選択](/images/xcode8_8.png)  

###Build Settings > Signing
先程、Build SettingsからTeamを選択する話をしましたが、よくよく見ると...  

* Provisioning Profile  
* Provisioning Profile (Deprecated)  

の2つがあることに気づいたのではないでしょうか？？  
これは、GeneralからProvisioning Profileを自動 or 手動で設定できるようになった弊害と考えられます。  
これまで、Provisioning Profileの名称とUUIDが紐付けられる形でXcodeプロジェクトに保存されていたため、Provisioning Profileを設定した状態でGitにアップすると、他の人がPullした際に、Provisioning ProfileにUUIDの値が表示されるということがありました。  
今後はTeamに開発メンバーとして含まれていれば、こういったことが解消されることでしょう。  

###Storyboard
続いて、筆者が目を惹かれたのはStoryboardです。  
これまでは特定のデバイスの形によらない正方形の形をデフォルトとしていました。  
これを右メニューからデバイスの形を選択することで変更していました。  

Xcode8からは下部に新しいメニューが表示され、そこから簡単に特定のデバイス型にStorybaordを変更することができます。  

![新しいStoryboard](/images/xcode8_9.png)  

###App Icons
さて、ここはそんなに大きな変更はありませんでした。  
強いて言えば、右メニューのUIがチェックボックス形式からセレクトボックス形式に変更になったくらいです。  

![App IconsのUI](/images/xcode8_10.png)  

因みに、Xcode8は正式にiOS7以前のサポートを排除しました。(Deployment Targetを見るとiOS8.0からの選択になっていることがわかります。)  
筆者が疑問なのは、上図のApp Iconsを見ると`iOS 5, 6`とか`iOS7 - 10`のように古いiOSのナンバリングが未だになぜ残っているのだとう...ということです。  
開発する上で特別気にする必要はないんでしょうが...  

###まとめ
さて、本日はXcode8の変更点を少しだけ見てみました。  
まだ筆者もXcode8で十分な数のサンプルアプリを開発できているわけではないため、見つけられていない新機能がたくさんあることでしょう。  
これから少しずつではありますが、役に立つ新情報を見つけていければと思います。  
と言ったところで本日はここまで。  


参考:  

* [Code Signing in Xcode 8](https://possiblemobile.com/2016/06/code-signing-xcode-8/)  




