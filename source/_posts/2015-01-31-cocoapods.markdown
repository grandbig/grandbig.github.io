---
layout: post
title: "自作ライブラリをCocoaPodsで管理しよう！"
date: 2015-01-31 22:54
comments: true
categories: ios CocoaPods
---

####自作ライブラリをCocoaPodsでインストールしたい
さて、本日はCocoaPodsについて書いてみます。  
最近、GitHubなどで公開されているiOS向けOSSのほとんどはCocoaPodsで管理されていることと思います。  
そんな中、『自分で開発したライブラリも皆に使ってもらうなら、カッコよくCocoaPodsでインストールしてほしい』といった思いも出てきますよね。  
よし！やってみよう！というのが本日の記事です。  

では、早速、その手順です。  
まずは、CocoaPodsで管理する対象の構造を下記に示します。  

```objective-c
Classes ━┳━━ jsonUtil
         ┃     ┣━━ jsonUtil.h
         ┃     ┗━━ jsonUtil.m
         ┃
         ┗━━ Validation
               ┣━━ Validation.h
               ┗━━ Validation.m
```

<!-- more -->

１．GitHubにファイルをアップロード  
OSSとして公開するのですから、[GitHub](https://github.com/)にアップしましょう。  
アカウントを作成して、新レポジトリを作成しましょう。  
詳しくは[GitHub（ギットハブ）の使い方：登録編](http://www.atmarkit.co.jp/ait/articles/1407/22/news019.html)を参考にしましょう。  

２．podspecファイルを作成  
ターミナルから公開したいファイルの階層に入り、podspecファイルを作成しましょう。  

```objective-c
cd <指定ファイルのディレクトリ>
pod spec create <公開ライブラリの名称>
```

３．podspecファイルを編集  
podspecファイルを編集して、GitHubとの連携などを設定しましょう。  
とにかく今回書いたpodspecファイルは下記です。  

```objective-c
Pod::Spec.new do |s|

<省略>

  s.name         = "CocoaPodsSampleLibrary"
  s.version      = "0.0.1"
  s.summary      = "Library for CocoaPods Test"

  s.description  = <<-DESC
                   A longer description of CocoaPodsSampleLibrary in Markdown format.

                   * Think: Why did you write this? What is the focus? What does it do?
                   * CocoaPods will be using this to generate tags, and improve search results.
                   * Try to keep it short, snappy and to the point.
                   * Finally, don't worry about the indent, CocoaPods strips it!
                   DESC
  
  s.homepage     = "https://github.com/grandbig/CocoaPodsSampleLibrary"

<省略>


  s.source       = { :git => "https://github.com/grandbig/CocoaPodsSampleLibrary.git", :tag => "#{s.version}" }

<省略>

  s.subspec 'jsonUtil' do |ss|
    ss.source_files = 'Classes/jsonUtil/*.{h,m}'
  end

  s.subspec 'Validation' do |ss|
    ss.source_files = 'Classes/Validation/*.{h,m}'
  end

<省略>

end
```

`s.source`にはCocoaPodsでのインストールをするために`:git => "..."`の部分にGitHubからCloneするたときに指定するURLを書きましょう。  

そして、CocoaPodsでインストールした後もフォルダ構成を保ちたいのであれば、単純に`s.source_files  = "Classes", "Classes/**/*.{h,m}"`と指定するのではなく、
`s.subspec 'jsonUtil' do |ss| 〜 end`と指定しましょう。  

さて、これで全ての対応が完了しました。  
実際に、CocoaPodsでインストールしてみましょう！  
思い描いた通りに自作ライブラリが追加されているはずです。  
といったところで今日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

