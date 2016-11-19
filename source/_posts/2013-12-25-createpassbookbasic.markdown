---
layout: post
title: "Passbookを使ってみよう！(超基礎)"
date: 2013-12-25 23:47
comments: true
categories: ios Passbook
---

###Passbook 超入門
今日はPassbookについて書いてみようと思います。この記事を書こうと思ったきっかけはネットでPassbookのPassの作成方法を探してみたところ、『いまいちわかりにくい...。画像で手順を見たい！！』という思いが生まれたからです。今後、日本でもPassbookの利用を盛り上げていきたいという意向もあります。だって触ってみると案外面白いんですから。  
では、早速説明していきましょう。  

<!--more-->

####キーチェーンアクセスに証明書を登録しよう
Apple Developerプログラムに登録されている前提で説明します。  
まず、**iOS Pass Type IDs**を新規作成します。右上の『＋』ボタンをクリックしましょう。  
![iOS Pass Type IDsを新規作成](/images/create_passbook_cert1.png)  
次に、**Pass Type ID Description**と**Identifier**を入力してContinueを選択して先に進みましょう。  
※ Identifierは必ず先頭に**pass.**がきます。  
![任意の文字列を入力](/images/create_passbook_cert2.png)  
そして、Registerをクリックすれば、**iOS Pass Type IDs**を新たに作成されたことを確認できます。  
![Registerをクリック](/images/create_passbook_cert3.png)  
![iOS Pass Type IDsを新規作成できました](/images/create_passbook_cert4.png)  
ここで終了ではありません。ここから**iOS Pass Type IDs**と**certSigningRequestファイル**をひもづける作業が必要となります。  
先ほど作成した**iOS Pass Type IDs**を選択してEditをクリックしましょう。  
![iOS Pass Type IDsを編集](/images/create_passbook_cert5.png)  
編集画面でCreate Certificate...を選択して、アップロードするファイルの説明書きを読んでContinueを選択したら、Choose File...をクリックして**certSigningRequestファイル**をアップロードしましょう！！  
![Create Certificateを選択](/images/create_passbook_cert6.png)  
![Continueでそのまま進む](/images/create_passbook_cert7.png)  
![Choose Fileを選択](/images/create_passbook_cert8.png)  
これで証明書が作成できました。Downloadしてキーチェーンアクセスに登録しましょう！(Chromeであればブラウザの下部にダウンロードファイルが表示されるのでダブルクリックしましょう)  
![証明書をDownloadしましょう](/images/create_passbook_cert9.png)  
![キーチェーンアクセスに登録完了](/images/create_passbook_cert10.png)  

####pass.jsonを編集しよう
証明書の準備が整ったら実際にPassbookのPassの中身を作成していきます。0から作るのはたいへんですし、いろいろとApple側で決められている制限に従う必要があるので、[Apple公式ダウンロードページ](https://developer.apple.com/downloads/index.action?name=Passbook)からサンプル(Passbook Materials)を落としてきましょう。  
dmgファイルを解凍すると中身のファイル構成は下記のようになっています。  
![Passbook Materialsの中身](/images/create_passbook_cert11.png)  

Passesの配下には**.pkpassファイル**と**.rawフォルダ**があります。PassbookのPassとして扱われるファイルが**.pkpassファイル**です。**.rawフォルダ**はPassを構成する素材が入っています。実は**.pkpassファイル**は**.rawフォルダ**を圧縮したものになっています。  

また、ファイルやフォルダの種類としてBoardingPass, Coupon, Event, Generic, StoreCardの5種類がありますが、これはPassbookの種類になります。これはレイアウトが異なるので最適なものを利用するようにしましょう。  

さて、ここからは実際にサンプルを編集することでオリジナルPassを作成してみたいと思います。  
先ほど解答したフォルダからEvent.rawフォルダを任意の場所にコピーしましょう。  
Event.rawフォルダ内のpass.jsonを編集します。  

```
{
	"formatVersion" : 1,
	"passTypeIdentifier" : "pass.com.apple.sample-event",
	"serialNumber" : "nmyuxofgnb",
	"teamIdentifier" : "A1B2C3D4E5",
```

必ず編集する必要があるのは**passTypeIdentifier**と**teamIdentifier**です。  
**passTypeIdentifier**はキーチェーンアクセスから表示されているPass Type IDを使いましょう。(Apple Developerサイトからも確認できます。) また、**teamIdentifier**はキーチェーンアクセスでPassbookの証明書を右クリックして『情報を見る』を選択してください。その中の組織に表示されている英数字を使います。  
![passTypeIdentifierの確認](/images/create_passbook_cert12.png)  
![teamIdentifierの確認](/images/create_passbook_cert13.png)  
これを先ほどのpass.json内に記載しましょう。  

####Passを作成しよう
さて、Passを作成するための素材は揃いました。最後に**.pkpassファイル**を作成するためのsignpassファイルを書き出します。  
手順としては先ほどダウンロードしたdmgファイル内にあったsignpassフォルダ内のsignpass.xcodeprojファイルをXcodeで起動します。これをBuildしてください。デフォルト設定のままであれば『/Users/＜ユーザー名＞/Library/Developer/Xcode/DerivedData』配下にsignpass.....(signpassの後はよくわからない文字列)が作成されています。もし、デフォルト設定を変更したせいか、見つからない場合はXcode > Preferences... > Locations > DerivedDataで設定されているパスを確認して下さい。  

該当フォルダを見ると、Build/Products/Debug配下にsignpassファイルが見つかるはずです。
ここまで来たら後は下記コマンドをターミナルで叩くだけ！
```
./signpass -p ＜パッケージするパスファイルのパス＞
```
パッケージするパスファイルのパスとは例えば、signpassファイルとEvent.rawフォルダが同じ階層にある場合は
```
./signpass -p ./Event.raw
```
となります。
これで**Event.pkpassファイル**が作成されているはずです。これにてPass作成完了です。  

####PassをWeb配布しよう
最後にWebでPassを配布する方法について解説します。  
とりあえず、ローカルPCからPassを取得してみたいと思います。これを実現するにあたって、『システム環境設定の共有にWeb共有がない』なんて方がいるかもしれないので、それは[こちら](http://d.hatena.ne.jp/sakura_bird1/20120804/1344055999)を参考に設定してください。  
pkpassファイルを取得するためにはhttpd.confにpkpassファイルの行を書き足す必要があります。  
※Apacheを使っている場合の説明です。  
```
AddType application/vnd.apple.pkpass    pkpass
```
AddTypeが書かれている<IfModule>タグ内に記載してください。  
apacheを再起動することをお忘れなく。  
これで晴れてPassをWebから取ってくることができます！

![Passのサンプル](/images/create_passbook_sample.png)  

細かいPassの編集については割愛しましたが、試してみていろいろとわかってきたらブログにも載せていきます。  

では本日はこの辺で！
