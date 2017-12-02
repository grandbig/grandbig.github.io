---
layout: post
title: "今更だけど試してみようUniversalLink"
date: 2017-12-02 18:16
comments: true
categories: ios swift
---

### UniversalLinkとは
UniversalLinkとはiOS9と共に登場したカスタムURLスキームに代わるアプリ起動支援の仕組みです。  
例えばどんなことができるかというと...  

アプリの宣伝用のLPへのURLリンクをタップした時にアプリがインストール済みであれば、  
Safariを開かずにアプリを起動する  

といったことができます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

どういったシーンで利用できるかというと...  

* 既にインストール済みのアプリであれば、宣伝用ページではなくアプリに飛ばしたい  
* 会員登録時に仮メールを送信して、メール内リンクをタップしたらアプリを起動させたい  
* キャンペーン期間中のみ、特定のリンクを踏むと、アイテムやクーポンがもらえる  

などが考えられます。  

### UniversalLinkの設定方法
では、具体的にどうすればUniversalLinkが利用できるのでしょうか。  

#### Apple Developer Program上での設定
Xcode上で設定すれば、Apple Developer Programから設定する必要はないとの記事も見かけましたが、  
未確認であること、いずれにしても必須設定であることから説明します。  

① Apple Developer Programにアクセス  
② `Certificates, Identifiers & Profiles` > `Identifier` > `App IDs` を選択  
③ 利用する `App IDs` を編集して `Associated Domains` を `Enabled` に変更  

![Apple Developer Program上の設定](/images/universallink_1.png)  

たったこれだけの設定でOKです。  

#### Xcode上での設定
ここからはアプリ側の設定を説明します。  
と言っても難しいことはありません。  

① 先程の手順で設定した `App IDs` が `Bundle Identifier` に設定されたプロジェクトを作成  
② `Capabilities` > `Associated Domains` を `ON` に変更  
③ `Domains` に飛ばしたいドメインを追加  

![Xcode上の設定](/images/universallink_2.png)  

このとき、 `Steps` の2行が `✔` 状態になっているかどうかも重要なポイントです。  

たったのこれだけの設定でOKです。  

#### サーバサイドの設定
最後にサーバサイドで必要な設定について説明します。  
サーバサイドの環境や設定次第では少々苦労することがあるかもしれません。  

① `apple-app-site-association` ファイルを作成する  

```javascript
{
  "applinks": {
    "apps": [],
      "details": [
        {
          "appID":"xxxxxx.com.hogehoge.AppSample",
          "paths":[ "*" ]
        }
      ]
    }
}
```

上記で重要なポイントは...  

* `json` 形式だが、拡張子に `json` はつけない  
* `apps` は必ず `[]` で中身は何も書かない  
* `appIDs` は **<Team ID>.<Bundle Identifier>** の形式で書く  
* `paths` はアクセス時に飛ばしたいパスの詳細設定を書く ( `"*"` は全て飛ばします )

になります。  
`Team ID` は `Apple Developer Program` 上から確認できます。  

![Team IDの記載場所](/images/universallink_3.png)  

② 作成したファイルを `https` サーバのドキュメントルートに配置する  
`http` サーバの場合は `apple-app-site-association` をTLS証明書で署名する必要がありますが、今回は割愛します。  

ドキュメントルートはWebサーバの設定ファイルから確認することができます。  

```javascript
// Apacheの場合
// httpd.conf

DocumentRoot /home/user/www/html
```

例えば、上記のように設定されているのであれば、 `/home/user/www/html` 配下に `apple-app-site-association` ファイルを置きましょう。  

③ Webサーバの設定を確認する  
UniversalLinkの機能を利用するにはWebサーバが下記設定になっている必要があります。  

* `apple-app-site-association` ファイルにリダイレクトすることなくアクセスできる  
* 署名していない場合は `Content-type` に `application/json` を指定する  
署名している場合は `Content-type` を `application/pkcs7-mime` に設定する  

```javascript
// Apacheの場合
// httpd.conf

# リダイレクトによるアクセス防止
RewriteRule ^/apple-app-site-association - [L]

# Content-typeの設定
<Directory /home/user/www/html>
...
  <Files apple-app-site-association>
    Header set Content-type "application/json"
  </Files>
</Directory>
```

ここまでできれば完成です。  

### 設定完了後の動作確認方法
さて、動作確認には以下2つの方法があります。

前提として、起動させたいアプリをインストールする必要があります。  

【動作確認方法】  
① Safariで直接URLにアクセス後、ページを上から下にスワイプすると、アプリの表示バーが出て来るので、それをタップする  
② メール等に記載されたURLをタップする  

①または②の方法でアプリが起動すればUniversalLinkの設定は完了したと言えます。  
※Appleが提供している公式ツールもあるのですが、テスト環境用サーバだとアクセスを制限している可能性があるので、上記方法が良いかなと思っています。  

### まとめ
さて、iOS11が登場して1ヶ月ほど経過しているにも関わらず、今更ながらUniversalLinkの設定方法について書いてみました。  
自分でいざ設定しようとすると案外ハマりどころがあったりするので、対応する機会がある方はぜひ試しに対応してみることをオススメします。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
