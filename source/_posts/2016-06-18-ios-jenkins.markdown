---
layout: post
title: "JekninsからGitHubにアップされたiOSプロジェクトをビルドしよう！"
date: 2016-06-18 22:57
comments: true
categories: ios jenkins
---

####Jenkins / GitHub / Xcodeを連携させよう！
さて、今日はJenkinsとGitHubとXcodeの連携について見ていきます。  
筆者はこれまで、手元の自身のMac端末でビルドして、Fabricでアプリを配信することが多かったのですが、最近、[Deploygate](https://deploygate.com/?locale=ja)を使う場面が出てきました。  
業界スタンダートな方法なのかわからないものの、JenkinsとDeploygateを連携させてJenkins上でビルドしたアプリをDeploygateから配信することができます。  
そういった筆者にとって新しい機会に恵まれたものの、触ってみた経験がないためわからないことが多々あったりします。  
そこで、今回はローカル環境でJenkins / GitHub / Xcodeを連携させて遊んでみようと思います。  

<!-- more -->

#####Jenkinsのインストール
まずは、JenkinsをHomebrewでローカル環境にインストールします。  
`brew install jenkins`を実行します。  

```objective-c
Password:
==> Downloading http://mirrors.jenkins-ci.org/war/2.9/jenkins.war
==> Downloading from http://ftp.yz.yamagata-u.ac.jp/pub/misc/jenkins/war/2.9/jenkins.war
######################################################################## 100.0%
==> jar xvf jenkins.war
==> Caveats
Note: When using launchctl the port will be 8080.

To have launchd start jenkins now and restart at login:
  brew services start jenkins
Or, if you don't want/need a background service you can just run:
    jenkins
==> Summary
🍺  /usr/local/Cellar/jenkins/2.9: 6 files, 66.4M, built in 59 seconds
```

Jenkinsをこれから使う機会も多いかもしれないので自動起動設定もしちゃいましょう。  

```objective-c
cp -p /usr/local/opt/jenkins/*.plist ~/Library/LaunchAgents
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist
launchctl start homebrew.mxcl.jenkins
```

本当に起動してるかな〜と心配な方は`ps aux | grep jenkins`を叩いちゃいましょう！  
すると下記のように表示されます。  

```
/usr/bin/java -Dmail.smtp.starttls.enable=true -jar /usr/local/opt/jenkins/libexec/jenkins.war --httpListenAddress=127.0.0.1 --httpPort=8080
```

#####Jenkinsの初期登録を完了させよう！
インストール完了したので、Jenkins上でジョブを作りましょう！  
早速、`http://localhost:8080`にアクセスしてください。  

すると、 **Unlock Jenkins** と大きく書かれた画面が表示されます。  
この画面の表示された指示に従って、管理者パスワードを入力しましょう。
(コマンドは画面に表示された通りにターミナルで叩けばOKです。)  

![初期画面](/images/jenkins-1.png)  

続いて、 **Customize Jenkins** 画面に遷移するので、  
**Install suggested plugins** を選択しましょう。  

![Customize Jenkins](/images/jenkins-2.png)  

そして、ユーザー登録をします。  

![ユーザー登録](/images/jenkins-3.png)  

これでJenkinsのトップ画面が表示されます。  
Xcode Pluginがまだインストールされていないので、インストールします。  

左メニューから「Jenkinsの管理 > プラグインの管理」を選択します。  

![プラグインの管理](/images/jenkins-6.png)  

検索ボックスでXcodeと入力してください。  
**Xcode integration** が見つかりますので、インストールしてください。  
これで必要なプラグインのインストールが完了しました。  

#####Jenkinsで新規ジョブを作成しよう！
左メニューから新規ジョブを作成します。  

![新規ジョブの作成](/images/jenkins-4.png)  

ジョブ名を入力して、「フリースタイル・プロジェクトのビルド」を選択して、OKを押します。  

![ジョブのベースを作成](/images/jenkins-5.png)  

まず、GitHubと連携させましょう。  
JenkinsのジョブにGitのRepository URLを設定します。  

![JenkinsでGitのRepository URLを設定](/images/jenkins-7.png)  

ただ単に設定すると403 Errorが出てしまうので、GitHub側を設定します。  
GitHubから取得したいRepositoryを選択します。  
Settings > Webhooks & services > Add Webhook を選択します。  
下記のように設定しましょう！  

![Webhooks & services](/images/jenkins-8.png)  

これでJenkins側の403 Errorが解消されるはずです。  

続いてビルド情報を入力していきます。  

ビルド手順の追加 > 「シェルの実行」を追加します  

```objective-c
#!/bin/bash -l
pod install
```

このタイミングで、`.bashrc`に`export LC_ALL="en_US.UTF-8"`を追加しておきましょう。  
(追加しないと『 **invalid byte sequence in US-ASCII** 』が後々発生します。)  

ビルド手順の追加 > 「Xcode」を追加します  
設定は下記のようにしてきます。  

【Code signing & OS X keychain options】  

![Code signing & OS X keychain options](/images/jenkins-9.png)  

【Advanced Xcode build options】  

![Advanced Xcode build options](/images/jenkins-10.png)  

|Xcode Schema File: |FacebookLoginSample|
|:-|:-:|
|Xcode Workspace File: |FacebookLoginSample|
|Xcode Project File: |FacebookLoginSample|
|Build output directory: |${WORKSPACE}/build|
　

#####Jennkinsでジョブを実行しよう！

これでジョブを実行してみましょう！  
左メニューから『 **ビルド実行** 』を選択します。  

![ビルド実行](/images/jenkins-11.png)  

ビルドを実行すると、ビルド履歴に新たに項目が追加されます。  
詳しいビルド状況を追いたければ、時間部分を選択します。  

![ビルド状況の確認](/images/jenkins-12.png)  

そして、画面遷移後の左メニューから『 **コンソール出力** 』を選択します。  

![コンソール出力](/images/jenkins-13.png)  

これで最後までログを追うことができます。  

####まとめ

さて如何でしたでしょうか？  
筆者も初めてローカル環境でJenkinsを立ててみましたが、意外とハマリポイントが多かったです。  
証明書周りでもう少し調べてみたいところもあるので、またわかったことがあればブログに書いていきたいと思います。  
と言ったところで本日はここまで。  

参考:  

* [OS XにJenkinsをHomebrewでセットアップする](http://qiita.com/makoto_kw/items/cbe93d4ebbc35f3b43fd)  
* [Jenkinsをインストールして使ってみよう](http://www.buildinsider.net/enterprise/jenkins/001)  
* [invalid byte sequence in US-ASCII](https://github.com/CocoaPods/CocoaPods/issues/639#issuecomment-11483748)  

