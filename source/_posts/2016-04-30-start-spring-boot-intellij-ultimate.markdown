---
layout: post
title: "快適なSpring BootプログラミングのためにIntelliJ IDEA Ultimate版を購入しよう！"
date: 2016-04-30 14:48
comments: true
categories: java springboot
---

####はじめに
筆者は決してJetBrains社の回し者ではありません笑  
最近、Spring Bootにチャレンジしているのですが、自宅ではvimを使っていました。  
なぜかって、IntelliJ IDEA Community版だとSpring Bootプログラミングの際に面倒なことが多いんですもの...。  
しかしながら、vimでSpring Bootプログラミングを続けるのもしんどいので、  
とうとうIntelliJ IDEA Ultimate版に手を出してしまったということなんです。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####IntelliJ IDEA Ultimate版とは
さて、早速、IntelliJ IDEA Ultimate版の何が良いかについて説明していきます。  
IntelliJ IDEAとは[JetBrains](https://www.jetbrains.com/)社が開発した多言語対応の統合開発環境のことです。  
Android StudioがIntelliJ IDEAを元に作られていることは割りと有名な話ですね。  

筆者が[Spring Bootを使い始めよう！](http://grandbig.github.io/blog/2016/04/16/start-spring-boot/)の記事内で使用していたIntelliJ IDEAはCommunity版だったのですが、  
もちろん、Community版でもプログラミングを書くことやGradleによるビルド実行は問題ありません。  
しかし、Community版では例えば下記のようなことに対応できません。  

* Spring Boot Framework関連の予測変換  
* Thymeleaf関連の予測変換  
* Spring Bootを利用したプロジェクトをIntelliJから新規作成可能  

これに対応したのが **IntelliJ IDEA Ultimate版** なのです。  
詳細は公式サイトから知ることができます。  

![IntelliJ 比較表](/images/start-spring-boot-intellij-ultimate1.png)  

また、価格ですが、個人版であれば、ビジネス版よりも比較的に安く購入することができます。  
1年単位での購入なのですが、使い続けるほどに安くなるという価格設定がエンジニア魂をくすぐります。  
下図のように、3年目以降は初年度の半額弱にまで安くなります。  
(Apple Developer Programも同じような仕組みにして欲しいですね笑)  

![IntelliJ IDEA Ultimate版の価格](/images/start-spring-boot-intellij-ultimate2.png)  

因みに、同ページに「All Products Pack」という商品もありますが、  
PHP, Ruby等々を無双したい方向けなので、筆者には今のところ不要です。  

####IntelliJ IDEA Ultimate版の購入と設定
IntelliJ IDEA Ultimate版の購入方法と設定について説明します。  
購入は非常に簡単です。  

1．[Buyページ](https://www.jetbrains.com/idea/buy/#edition=personal)で **Buy Now** をクリック  

![Buyページ](/images/start-spring-boot-intellij-ultimate2.png)  

2．CustomerページでEmailアドレスを入力してNEXTをクリック  

![Customerページ](/images/start-spring-boot-intellij-ultimate3.png)  

3．Emailアドレスと国を確認してPROCESS AS NEW CUSTOMERをクリック  

![Customerページ2](/images/start-spring-boot-intellij-ultimate4.png)  

4．Customer Infoページで必要事項を入力してPLACE ORDERをクリック  

![Customer Infoページ](/images/start-spring-boot-intellij-ultimate5.png)  

5．JetBrains社から届いたメールの添付ファイルを開封して、リンクをクリックしてActivationページへ遷移  

![JetBrains社から届いたメールの添付ファイル](/images/start-spring-boot-intellij-ultimate6.png)  

6．Activationページを確認  

![Activationページ](/images/start-spring-boot-intellij-ultimate7.png)  

ここまでで、購入完了したことを確認できます。  
続いて設定に必要な工程について説明します。  

7．ActivationページでDownload activation codeをクリック  

![Activationページ2](/images/start-spring-boot-intellij-ultimate8.png)  

8．IntelliJ IDEA Ultimate版をDownLoad  

![Ultimate版をDownLoad](/images/start-spring-boot-intellij-ultimate9.png)  

9．Activation codeをIntelliJ IDEA Ultimate版に登録  

DownLoadしたIntelliJ IDEA Ultimate版を起動して、先ほどDownLoadしたActivation codeを設定しましょう。  

![IntelliJ IDEA Ultimate](/images/start-spring-boot-intellij-ultimate10.png)  

以上で設定も完了です。  

####IntelliJ IDEA Ultimate版でSpring Bootを始めよう！
では、IntelliJ IDEA Ultimate版でSpring Bootを利用する新規プロジェクトを作成しましょう。  

1．IntelliJ IDEA Ultimate版を起動してCreate New Projectをクリック  

![Create New Project](/images/start-spring-boot-intellij-ultimate11.png)  

2．Spring Inilializrを選択  

![Spring Initializr](/images/start-spring-boot-intellij-ultimate12.png)  

3．各種名称や項目を設定  

![各種名称や項目の設定](/images/start-spring-boot-intellij-ultimate13.png)  

4．必要なDependencyを設定  

![必要な名称や項目の設定](/images/start-spring-boot-intellij-ultimate14.png)  

5．プロジェクト名と保存場所を設定  

![プロジェクト名と保存場所を設定](/images/start-spring-boot-intellij-ultimate15.png)  

6．Gradleの設定の確認  

![Gradleの設定の確認](/images/start-spring-boot-intellij-ultimate16.png)  

以上で新規プロジェクトが作成できました。  
プロジェクトを実行してみましょう。  

![プロジェクトの実行](/images/start-spring-boot-intellij-ultimate17.png)  

さて、如何でしたでしょうか？  
非常に簡単に導入できますし、使い勝手も良いので、これからSpring Bootプログラミングを始める方は検討してみてはいかがでしょうか？  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
