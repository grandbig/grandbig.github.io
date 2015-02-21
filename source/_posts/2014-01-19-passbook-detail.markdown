---
layout: post
title: "Passbook webServiceURLについて"
date: 2014-01-19 21:30
comments: true
categories: ios Passbook
---

###Passbook webServiceURLの設定で躓いたところ
さて、最近はPassbookばかり触っているので、3回連続でPassbookについてです。筆者が躓いたところをピックアップして記載していきます。  
具体的には  
・webServiceURLに指定するURL  
・Passbookを追加/更新/削除したときに自動で投げてくるパラメータ  
についてです。

<!--more-->

####webServiceURLについて
PassbookではwebServiceURLを設定するところで躓きました。使いきり&更新や削除などを感知しないパスであれば、何も気にせず適当なURLを設定して構いません。しかし、パスの追加/更新/削除を感知して、ユーザ固有のアクションを起こさせたいという狙いがある場合は、きちんと考えなくてはなりません。  

結論から言うと、**http**ではなく**https**から始まるURLを指定しなくてはなりません。また、自己証明書で試した結果、通信ができませんでした...。これは筆者の設定が悪いのかどうか不明なところなんですが、きちんとした証明書で**https**通信ができるサーバで試した結果、通信できました。ここについてはネット上ではっきりと書かれているサイトがなかったため真実はわからないのですが、少なくとも筆者は自己証明書では失敗したということをお伝えします。  

因みに筆者は今回、Windows AzureでWebサイトを作成して活用しました。Webサイトではhttpでもhttpsでもアクセスできるように設定されているとのことで非常に使い勝手が良かったです。  

また、Apple iOS Developerであれば、開発者だけができる設定でwebServiceURLにhttpのURLを設定しても通信することができます。次の章で説明する内容を実際に見たい場合は余分なお金のかからないhttpサーバで試してみると良いでしょう。設定は下記のようにできます。  
![設定からhttpを許可する](/images/httpAllow.png)  

####Passbookを追加/更新/削除したときに自動で投げてくるパラメータ
続いて、Passbookをユーザが受け取ってからwebServiceURLに指定したサーバに投げてくるパラメータについて説明します。パラメータが送られるパターンは下記です。  
・Passbookを受け取って**追加**ボタンを押したとき  
・Passbookを手動/自動で**更新**したとき  
・Passbookを**削除**したとき  

それぞれについて具体的に説明します。  
#####Passbookを受け取って追加ボタンを押したとき  
追加ボタンを押した瞬間に自動的にPOSTで通信されます。  
その際、投げられるURLは
```
/webServiceURLで指定したURL/v1/devices/デバイスライブラリID/registrations/パスタイプID/シリアルナンバー
```
となります。
このURLもしくはreq.paramsから**デバイスライブラリID**, **パスタイプID**, **シリアルナンバー**を取得することができます。また、req.headersに**authorization**の値が入っています。  

デバイスライブラリIDはデバイスによって異なり、さらに新規パス取得の際には毎回異なる値です。パスタイプIDは開発者が発行するPassbookの証明書の.passを外した部分のことです。シリアルナンバーはパス毎に発行すべきものです。例えば、忘年会のチケットを配布する場合、そのチケットでは同じシリアルナンバーを使います。(ユーザごとにユニークな値をふるべきではありません。)authorizationはユーザ毎にユニークな値を持たせた方が使い勝手が良いと思われます。また、MD5などで暗号化した方がセキュリティ的には良いでしょう。  


#####Passbookを手動/自動で更新したとき
Passbookアプリでパスの裏面を上から下にスクロールして手動で更新したときなどではGETで通信されます。  
その際に投げられるURLは
```
/webServiceURLで指定したURL/v1/passes/パスタイプID/シリアルナンバー
```
となります。
更新の場合は**パスタイプID**, **シリアルナンバー**, **authorization**を取得することができます。  
また、更新処理のときにはログ用にPOSTで
```
/webServiceURL/v1/log
```
が投げられます。  
これを用いてログをDBに格納するのも良いでしょう。  

#####Passbookを削除したとき
最後はパスを削除した場合です。このときはDELETEで通信されます。  
その際、投げられるURLは
```
/webServiceURLで指定したURL/v1/devices/デバイスライブラリID/registrations/パスタイプID/シリアルナンバー
```
となります。  

因みに筆者の場合は、Node.jsでExpressモジュールを使用しているので、下記のように書いています。  
webServiceURLで末尾に/sendParamというパスをつけた場合です。  
```
// Passbookを受け取って追加ボタンを押したとき
app.post('/sendParam/v1/devices/:deviceLibraryID/registrations/:passTypeID/:serialNumber', pass.createData);
// Passbookを手動/自動で更新したとき
app.get('/sendParam/v1/passes/:passTypeID/:serialNumber', pass.updateData);
// ログ用
app.post('/sendParam/v1/log', pass.logData);
// Passbookを削除したとき
app.delete('/sendParam/v1/devices/:deviceLibraryID/registrations/:passTypeID/:serialNumber', pass.deleteData);
```

大体Passbookのことはわかってきたのですが、まだまだ躓くところも多く勉強です。本日はここまで。

