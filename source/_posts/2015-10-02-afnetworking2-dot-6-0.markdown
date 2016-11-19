---
layout: post
title: "AFNetworking 2.6.0でやってみよう自己証明書でのSSL通信"
date: 2015-10-02 22:21
comments: true
categories: ios
---

####AFNetworking2.5.xまでの自己証明書でのSSL通信
これまで[AFNetworking](https://github.com/AFNetworking/AFNetworking)2.5.xを使ってきた筆者ですが、開発中は当然のことながら自己証明書を利用することがあります。  
まずはこれまでの書き方を`AFHTTPSessionManager`を例に確認してみましょう。  

```objective-c
// AFHTTPSessionManagerをインスタンス化
AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
manager.securityPolicy.allowInvalidCertificates = YES;
manager.responseSerializer.acceptableContentTypes = [NSSet setWithObjects:@"text/plain", @"application/json", nil];

...

```

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

そうです。  
自己証明書を許可するのに、`manager.securityPolicy.allowInvalidCertificates = YES;`と書いていました。  

しかし、2015/08/18に **2.6.0** が登場した途端、ネット上で大騒ぎに...笑。  
[2.6.0のRelease Note](https://github.com/AFNetworking/AFNetworking/releases)を見てみると、 **Full Certificate Chain Validation has been removed from AFSecurityPolicy.** なんて一言が...。  

いろいろと困ったのですが、筆者の場合は次のやり方でうまくいきました。  

####AFNetworking2.6.0での自己証明書 SSL通信
はい。2.6.0では下記のように書きました。  

```objective-c
// AFHTTPSessionManagerをインスタンス化
AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
manager.securityPolicy.allowInvalidCertificates = YES;
manager.securityPolicy.validatesDomainName = NO;
manager.responseSerializer.acceptableContentTypes = [NSSet setWithObjects:@"text/plain", @"application/json", nil];

...

```

何が変わったかお気づきでしょうか？  
そうです。  
`manager.securityPolicy.validatesDomainName = NO;`を追加したのです。  

####まとめ
iOS9からATS設定が追加されたこともあり、HTTPSが推奨されています。  
しかも、正式な証明書が開発・商用関係なく使われていくことでしょう。  
なので、こういった更新は納得感ありますが、なかなか開発者には辛いところもありますよね。  
とは言え、AppleやGoogleが決めていくことにはついていった方が良いよな〜といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
