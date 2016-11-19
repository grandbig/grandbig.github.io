---
layout: post
title: "Xcodeが正常かチェックしましょう！"
date: 2015-09-23 13:35
comments: true
categories: xcode
---

####今、話題のXcodeGhostってなに！？
中国で出回っている改ざんされたXcodeのことだそうです。  
App Storeから一般公開向けのXcodeをダウンロードしたり、Apple DeveloperサイトからDeveloper向けの最新版をダウンロードするのであれば、特に何の問題もないでしょう。  
『いや、実はどこどこから...』なんて方がいたら、早速チェックしてみましょう。  

ユーザとして気をつけるのは、怪しいアプリはもちろんのこと、話題として上っているアプリはきちんと対応が完了しているのか確認してみるくらいでしょうか。  
Appleも感染アプリの削除に全力で取り組んでいるでしょうし、様子を見るしかなさそうですね。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####Xcodeの正常チェックの方法
さて、本日、Appleから以下の内容のメールが飛んできました。  

We recently removed apps from the App Store that were built with a counterfeit version of Xcode which had the potential to cause harm to customers. You should always download Xcode directly from the Mac App Store, or from the Apple Developer website, and leave Gatekeeper enabled on all your systems to protect against tampered software.  

When you download Xcode from the Mac App Store, OS X automatically checks the code signature for Xcode and validates that it is code signed by Apple. When you download Xcode from the Apple Developer website, the code signature is also automatically checked and validated by default as long as you have not disabled Gatekeeper.  

Whether you downloaded Xcode from Apple or received Xcode from another source, such as a USB or Thunderbolt disk, or over a local network, you can easily verify the integrity of your copy of Xcode. Learn more.  

で、最後の **Learn more** をクリックすることで専用サイトで方法を教えてくれます。  

コマンドは下記です。  
`spctl --assess --verbose /Applications/Xcode.app`

実行してから完了までに数分かかりますが、問題なければ、  

```
/Applications/Xcode.app: accepted
source=Mac App Store
```

もしくは、  

```
/Applications/Xcode.app: accepted
source=Apple
```

あるいは、  

```
/Applications/Xcode.app: accepted
source=Apple System
```

といった結果が得られるはずです。  
App Storeからダウンロードしたか、AppleのDeveloperサイトからダウンロードしたかで異なります。  

因みに、通常、正規のルートでダウンロードしてきた場合は、自動的にGatekeeperがチェックしているそうです。  
意図的にGatekeeperをdisableにしていない限り。  

Gatekeeperは『システム環境設定 > セキュリティとプライバシー > 一般 > ダウンロードしたアプリケーションの実行許可』から変更できるそうです。  
それがGatekeeperとか意識したことなかったですけど...。  
(ここで、すべてのアプリケーションを許可しているとGatekeeperがOFFだとか...。)  

筆者は念のため、調べて問題ないことを確認しましたが、気になる方は試してみてはいかがでしょうか。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
