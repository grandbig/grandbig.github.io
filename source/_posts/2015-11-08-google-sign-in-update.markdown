---
layout: post
title: "Google Sign-In SDK for iOSがアップデートされました！"
date: 2015-11-08 01:31
comments: true
categories: Google ios
---

###Google Sign-In SDK for iOSを利用する理由
本日は再び、Google Sign-In SDK for iOSについてお話したいと思います。  
筆者は業務でも積極的にGoogle Sign-In SDK for iOSを使っています。  
理由は、  

* [Google](https://itunes.apple.com/jp/app/google/id284815942?mt=8), [Google+](https://itunes.apple.com/jp/app/google+/id447119634?mt=8)アプリを入れているユーザはアカウントを入力しなくてもOAuth認証が可能  
* クライアント側で手軽にOAuth認証を実装できる  

といった2つがあります。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

前者は特にユーザの手間を省けるので、非常に有用だと考えています。  
例え、1つの端末内で複数のアカウントを使っていても、ユーザが選択できる形でOAuth認証をすることができます。  

![OAuth認証](/images/google-sign-in-update-1.png)  

もちろんGoogle, Google+共にインストールされていない端末であったとしても、メールアドレスとパスワードを入力する従来の方法で問題なくOAuth認証を通すことができます。  

後者は以前紹介した手順([Google Sign-In SDK for iOSを使ってみよう！](http://grandbig.github.io/blog/2015/09/13/google-sign-in/)を参照のこと)を踏むだけで、簡単に実装することができます。  
OAuth認証で得られるトークン(アクセストークンやIDトークン)は端末ローカルで保存することも多いため、サーバサイドよりもクライアントサイドでOAuth認証を通す方が良い場合もあります。  
あとは、サーバを自身で持てない場合もクライアントサイドで処理が済むので、非常に良いですね。  

###Google Sign-In SDK for iOSがアップデートされて、さらに使いやすくなりました！
上記で説明した通り、非常に実装が簡単でユーザにとっても利益のあるSDKではあるものの、微妙に使いにくい側面も持っていました。  
それは、IDトークンを取得するタイミングが`[[GoogleSinIn sharedInstance] signIn];`と`[[GoogleSignIn sharedInstance] signInSilently];`の2つのみだったからです。  
IDトークンはアクセストークンと同じく、有効期限が **発行されてから1時間** と定義されているため、IDトークンを利用する場合は割りと頻繁に更新情報を取得する必要があります。  
しかし、先ほど2つのメソッドはアプリ起動後に **ユーザがログイン済みか否かを判断した上で実行するメソッド** であるため、頻繁に利用するシーンには不向きと言えます。(アプリ初回起動後とそれ以外との判断を自身で定義したフラグなどで管理する必要があるため)  

それが先日公開されたv2.4.0から([Release Notes](https://developers.google.com/identity/sign-in/ios/release)を参照のこと)変更が加わりました。  
具体的には、  

* `- (void) getAccessTokenWithHandler:`がdeprecatedとなり、`- (void) getTokensWithHandler:`が代わりに追加  
* `- (void) refreshAccessTokenWithHandler:`がdeprecatedとなり、`- (void) refreshTokensWithHandler:`が代わりに追加  

の2点です。  

これまでアクセストークンを取得するために利用していた`getAccessTokenWithHandler`ではアクセストークンのみを返却していましたが、`getTokenWithHandler`では`GIDAuthentication`オブジェクトごと返却してくれるようになったため、アクセストークンだけでなく、IDトークンも取得することができるようになりました。  

また、`refreshAccessTokenWithHandler`ではアクセストークンのみをリフレッシュするものでしたが、`refreshTokenWithHandler`ではアクセストークンとIDトークンをリフレッシュしてくれるようになりました。  

IDトークンはアクセストークンと同じ1時間だけ有効なトークンなので、自然と更新タイミングはアクセストークンと同じになります。  
アクセストークンのみを使うのであれば、`getAccessTokenWithHandler`と`refreshAccessTokenWithHandler`で良かったのですが、IDトークンも使うとなると`getTokensWithHandler`および`refreshTokensWithHandler`でないと成り立ちません。  

###OAuth認証を通して取得する各種トークンについて
Google OAuth認証では様々なトークンが飛び交います。ここではそれぞれのトークンの利用用途について少し触れておきたいと思います。  

* one-time code  
これはアクセストークンを取得するための一時的なトークンです。  
今回のSDKとは直接からみませんが、[
Google Sign-In for server-side apps](https://developers.google.com/identity/sign-in/web/server-side-flow?hl=ja)に書かれている通りで、サーバサイドでアクセストークンを取得するために利用します。  
* access-token  
これは各種APIを利用するために必要なトークンです。  
API実行時にヘッダーにこのトークンを渡すことで、認証されたことを伝えることができます。(ヘッダーに入れていない場合はunauthorized errorが発生します。)  
* refresh-token  
これはアクセストークンをリフレッシュするために利用するトークンです。  
アクセストークンは有効期限が1時間なので、継続的にAPIを実行するためには、きちんとアクセストークンをリフレッシュする必要があります。  
* id-token  
JWT(Json Web Token)と呼ばれるデータ形式で、ユーザのユニークな識別子が入っています。  
各種サービス間でのID連携するときに利用します。  
例えば、GREEにログインするときにYahoo!アカウントを使ってログインした場合に、ユーザの基本情報(名前やメールアドレス)が自動で入力されていることがあると思います。  
そういった連携を組み込みたい場合に使います。  

今回、id-tokenを調べる上で、OpenID Connectの考え方に触れました。これまで全く気にしてこなかった内容なのですが、かなり大事であることがわかりました。  
これについてもどこかで記事を書ければ良いなと思います。  

と言ったところで本日はここまで。  

参考:  
* [Google アカウントの認証を OpenID から OpenID Connect に移行する方法](http://webos-goodies.jp/archives/how_to_migrate_from_openid_to_openid_connect.html)  
* [OpenID Connect 入門 〜コンシューマーにおけるID連携のトレンド〜](http://www.slideshare.net/kura_lab/openid-connect-id)  
* [OAuth 2.0 の Response Type 全パターン](http://oauth.jp/blog/2015/01/06/oauth2-multiple-response-type/)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
