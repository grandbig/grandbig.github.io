---
layout: post
title: "超簡単Windows AzureでのWebサイト構築"
date: 2015-12-13 02:07
comments: true
categories: web
---

####Windows AzureでWebサイトを構築しよう
さて、本日はWinsows Azureを使って簡単にWebサイトを構築したいと思います。  
筆者は数年ぶりにWindows Azureを触ったため、UI・UXが結構変わっているなという印象を受けました。  

Webアプリケーションを開発するとき、ページビューのみであれば、ローカル環境で十分でしょう。  
しかし、位置情報やBeaconなどのIoT関連のアプリケーションを開発する場合、ローカル環境のWebページを閲覧できないこともあるでしょう。  
そんなときはどうしてもサーバが欲しくなりますね。  
筆者はWindows Azureをオススメしたいと思います。(AWSは使ったことないけど...)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

それでは早速、簡単にWebサイトを構築する方法について説明しましょう。  

１. ポータルサイトからサーバを新規作成  

![サーバを新規作成](/images/windows-azure-1.png)  

２. Web+モバイル > Web Appを選択  

![サーバを新規作成２](/images/windows-azure-2.png)  

３. Webアプリ名を決めて作成  

![サーバを新規作成３](/images/windows-azure-3.png)  

４. デプロイを待つ  

![サーバを新規作成４](/images/windows-azure-4.png)  

５. サーバ詳細画面に遷移  

![サーバの新規作成が完了](/images/windows-azure-5.png)  

６. サーバのデプロイ完了通知を受け取る  

![サーバのデプロイ完了通知](/images/windows-azure-6.png)  

７. リソースの設定  

![サーバにリソースを設定](/images/windows-azure-7.png)  

※筆者の場合はGitHubのリポジトリと同期させます。  

８. リポジトリの同期  

![リポジトリの同期](/images/windows-azure-8.png)  

９. サーバ環境の完成  

![サーバ環境が完成しました](/images/windows-azure-9.png)  

本当にたったのこれだけでWebサイトを構築できるのです。  
Node.jsなどをインストールしてAPサーバを構築する場合は少々手間がかかりますが、Webサーバを構築するだけなら簡単です。  
ぜひ機会があれば使ってみて欲しいものです。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

