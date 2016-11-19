---
layout: post
title: "今一度確かめようiOSにおけるHTML5 APIでの位置情報の取得"
date: 2015-12-23 00:00
comments: true
categories: ios web
---

####はじめに
こちらはiOS Second Stage Advent Calendar 23日目の記事です。  

筆者はこれまで2年と少し技術ブログを投稿してきましたが、Advent Calendarへの投稿はこれが初めてとなります。  
どうぞよろしくお願いします。  

今回お話させて頂く内容はiOSアプリでの位置情報の取得に関することです。  
ほとんどの開発者の方は`CoreLocation.framework`を用いて位置情報の取得をされると思いますが、HTML5の`Geolocation API`を使っても位置情報の取得をすることができます。  
とは言え、なぜわざわざiOSのパートで **HTML5** による位置情報取得の話をするかと言うと1つ理由があるんです。  
その理由を紹介しつつ、現代のiOSによるHTML5の`Geolocation API`での位置情報取得について話したいと思います。  

<!-- more -->

####2012年当時のGeolocation APIでの問題について
実は筆者は3年ほど前に、iOSアプリに **『HTML5 Geolocation APIによる位置情報取得機能』** を実装する機会がありました。  
そのアプリは2011年途中から開発が始まり、その間にiOSのバージョンも4.x→5.x→6.xと移り変わってきました。  
iOS 4.x〜5.xの際は順調に進んでいたのですが、2012/09/19にiOS6.0が公開されるやいなや状況が一変しました。  
というのもiOS6.0ではHTML5の`Geolocation API`による位置情報の取得に問題があったからなのです。  

そもそも、HTML5の`Geolocation API`は下記のように使います。  

```javascript
navigator.geolocation.getCurrentPosition(function(position) {
	// 位置情報の取得に成功した場合
	var lat = position.coords.latitude,
		lng = position.coords.longitude,
		acc = position.coords.accuracy;
}, function(error) {
	// 位置情報の取得に失敗した場合
	switch(error.code) {
		case 1:
			alert("位置情報が許可されていません。");
			break;
		case 2:
			alert("デバイスの位置がわかりません。");
			break;
		case 3:
			alert("アイムアウトしました。");
			break;
	}
}, {
	// オプションの設定
	enableHighAccuracy: true,
	timeout: 6000,
	maximumAge: 60000
});
```

特に見て欲しいのが、オプションの設定の **maximumAge** です。  
**maximumAge** は位置情報の有効期限を示す設定なのですが、iOS6.xでは **maximumAge** を0で設定しないと、継続的な位置情報の取得ができませんでした。  
これはあくまでも一例なのですが、iOS6.0が出てきた当初は、下記のように悲痛な叫びが散見していました。  

* [iOS6でGeolocation APIが動かない？](https://discussionsjapan.apple.com/thread/10126475?tstart=0)  
* [Geolocation doesn't work with iOS6 web apps!](https://discussions.apple.com/thread/4313850?start=30&tstart=0)  

当時はGeolocation APIでの位置情報は利用に耐えないということで見送りましたが、現在のiOS9におけるGeolocation APIでの動作はどうなっているのでしょうか？  

####現代のiOSでのGeolocation APIはどうなっている？
さて、iOS9.xでは果たして`Geolocation API`の挙動はどうなっているのでしょうか？  
今回はこの挙動を確かめるために下記機能を持ったアプリを開発し、実験を試みました。  

【アプリの機能】  

* `Geolocation API`による位置情報取得機能を実装  
* `CoreLocation.framework`による位置情報取得機能を実装  
* 上記2つの位置情報をローカルストレージに保存(`RealmSwift`を利用)  

WEBフロントのソースコードは[GitHubのGeoLocationSample](https://github.com/grandbig/GeoLocationSample)に公開しました。  
iOSアプリのソースコードは[GitHubのGeoLocation_iOS](https://github.com/grandbig/GeoLocation_iOS)に公開しました。  

【実験したい内容】  

* 開発アプリを用いて幾つか位置情報を取得する  
* `Geolocation API`で取得した位置情報の評価(キャッシュの有無を調査)  
* `Geolocation API`と`CoreLocation.framework`で取得した値を比較  

【結果】  

実験結果を一部抽出してGoogleマップにプロットしてみました。  
![結果のプロット](/images/geolocation_plot.png)  

* `Geolocation API`と`CoreLocation.framework`では精度のに大きな違いが出ないことがほとんど  
* 地下鉄の駅に到着した際に取得するとWi-Fiのアクセスポイントをひろうため、精度に違いが出ないことが多い  
    * 精度が65 [m] の場合、Wi-Fiのアクセスポイントをひろっています。  
* `Geolocation API`はiOS9ではキャッシュの有効・無効が機能していた  

因みに、AppleがiOS9からATS設定を組み込んだように、世の中ではHTTPSを使うことが当たり前になってきています。  
(というよりも商用サービスでHTTPを使わないことが当たり前になってきていると言いますか...)  

実は、`Geolocation API`でも下記のような警告が出ました。  

```
getCurrentPosition() and watchPosition() are deprecated on insecure origins, and support will be removed in the future. You should consider switching your application to a secure origin, such as HTTPS. See https://goo.gl/rStTGz for more details.
```

####それぞれの使いどころは！？
iOS9での実験を経て、精度として問題がないことがわかりました。  
ではどんなときに`Geolocation API`を使って、どんなときに`CoreLocation.framework`を使えば良いのでしょうか？  
筆者はざっくりと下記のパターン分けを考えています。  

【`Geolocation API`を使う場合】  

* ネイティブでの実装よりもWEBでの実装に慣れている場合  
* 位置情報の利用シーンがアプリのフォアグラウンド起動時のみの場合  
* 現在地の位置情報を1回取得すれば良い場合(取得までの時間がかからない)  

【`CoreLocation.framework`を使う場合】  

* 位置情報の利用シーンがアプリのバックグラウンド起動時も含む場合  
* 位置情報の連続的な取得が必要な場合
* 現在地の位置情報を1回取得する際に多少時間がかかっても良い場合(`requestLocation`では約10秒かかる)  

もちろん、iOSに限った話ですし、Androidでの挙動も確かめる必要があります。  
旧Androidの場合、WebViewがAndroid OSに依存しているため、`Geolocation API`の対応具合にバラつきがある可能性もあります。(それこそ、バグを含んでしまっている場合、ユーザがOSバージョンアップしないと解決できないことにも成りかねません。)  
とは言え、AndroidもWebViewがOSから分離されましたし、旧Androidであっても、CROSSWALKを使う方法もあるので、やりようはありますし、HTML5 APIの実用性もどんどん上がっていくことでしょう。  
もしかしたら、これから益々、ネイティブとハイブリッドの使い分けを考える時代になるかもしれません。  

####まとめ
久々に見てみるといろいろ変わってるな...。  
といったところでiOS Second Stage Advent Calendar 23日目の記事は終了となります。  
ありがとうございました！  
(また来年、帰ってくるかも!?)  
