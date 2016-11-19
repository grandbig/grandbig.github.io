---
layout: post
title: "Spring Bootを使い始めよう！"
date: 2016-04-16 14:43
comments: true
categories: springboot java
---

####Spring Bootとは
本日はこれまでと全く趣向を変えて、 **Spring Boot** について書きたいと思います。  
[Spring Boot](http://projects.spring.io/spring-boot/)とはSpringプロジェクトが提供している多くのFrameworkを統合した、  
簡単にWebアプリケーションを作成できるように設計されたFrameworkです。  

開発者はSpringプロジェクト内で用意された様々なFrameworkを意識する必要がありません。  
Spring Bootとは長年ブラッシュアップされ、肥大化してきたSpringプロジェクトへの参入ハードルを劇的に下げるものと言えるでしょう。  

なんて言いつつも、  
筆者はJavaによるWebアプリケーション開発はほとんどやったことがありません。  

何はともあれ、まずは触れてみたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####Spring BootによるWeb Application Projectを作成しよう！
最も簡単な方法について説明します。  

1．[Spring Initializr](https://start.spring.io/)にアクセス  
Projectの自動生成ツールサイトがあるので、アクセスして各種項目を設定しましょう。  

* Project: GradleおよびMavenから選択できます  
* Spring Boot: バージョンを選択できます  
* Group: パッケージ名を指定できます  
* Artifact: メインファイル名を指定できます  
* Dependencies: 初期状態で追加されるdependenciesを指定できます  

筆者は下記のように指定しました。  

![Spring Initializr](/images/start-spring-boot1.png)  

2．Generate Projectボタンをクリック  
下記のようなProjectが生成されます。  
(下記は筆者が指定した設定項目で生成されたProjectです。)  

```java
firstApp
  ┣━━ build.gradle
  ┣━━ gradlew
  ┣━━ gradlew.bat
  ┣━━ gradle
  ┃    ┗━━ wrapper
  ┃          ┣━━ gradle-wrapper.jar
  ┃          ┗━━ gradle-wrapper.properties
  ┗━━ src
       ┣━━ main
       ┃    ┣━━ java
       ┃    ┃    ┗━━ com
       ┃    ┃         ┗━━ example
       ┃    ┃               ┗━━ FirstAppApplication.java
       ┃    ┗━━ resources
       ┃         ┗━━ application.properties
       ┗━━ test
            ┗━━ java
                 ┗━━ com
                      ┗━━ example
                            ┗━━ FirstAppApplicationTests.java
```

3．`gradle bootRun`コマンドを実行  
firstApp直下で`gradle bootRun`コマンドを実行します。  
下記のような実行結果が出るはずです。  

![実行結果](/images/start-spring-boot2.png)  

これは生成されたメインファイルでのソースが下記のようになっているためです。  
`main`メソッド内で実行している処理はたったの1行で、  
これはSpring Bootによるアプリケーションを起動する処理となっています。  

ここで設定している`@SpringBootApplication`アノテーションは  
`@Configuration`, `@EnableAutoConfiguration`, `@ComponentScan`の3つの設定と同義になります。  

* `@Configuration`: Beanを含むJavaConfig用のクラスに付与する  
* `@EnableAutoConfiguration`: 様々な設定を自動で有効にする  
* `@ComponentScan`: クラスの自動スキャンを可能にする  

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class FirstAppApplication {

	public static void main(String[] args) {
		SpringApplication.run(FirstAppApplication.class, args);
	}
}
```

因みに、初期状態で`sourceCompatibility`および`targetCompatibility`が **1.8** となっています。  
よって、実行するためにはJDKバージョン **1.8** が必要になります。  
(インストールは[こちら](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)から)  

まずは最も簡単な導入が完了したようです。  

####IntelliJ IDEAにimportしてみよう
ターミナルからプロジェクトの開発を続けても良いのですが、  
ターミナルに慣れていない方は統合開発環境ツールで開きたいかもしれませんね。  
複雑なプロジェクトになればなるほど、ターミナルでの開発は辛くなるかもしれません。  

ということでIntelliJ IDEAに先ほどのプロジェクトをimportしてみましょう。  

1．IntelliJ IDEAを起動して、importを選択  
![import](/images/start-spring-boot3.png)  

2．importしたいDirectoryを選択  
![Directoryの選択](/images/start-spring-boot4.png)  

3．Createの方を選択  
![Createの方を選択](/images/start-spring-boot5.png)  

4．Project名と保存場所を確認  
![Project名と保存場所を確認](/images/start-spring-boot6.png)  

5．流れに任せてNextを続ける  
![ソースファイル一覧](/images/start-spring-boot7.png)  

![ライブラリの確認](/images/start-spring-boot8.png)  

![ライブラリの確認](/images/start-spring-boot9.png)  

![JDKを選択](/images/start-spring-boot10.png)  

6．一通り作業が終わったらFinishを選択  
![Finishを選択](/images/start-spring-boot11.png)  

7．Intellij IDEAのターミナルで`gradle bootRun`を実行  
![コマンドを実行](/images/start-spring-boot12.png)  

続いて、最も簡単なIntelliJ IDEA上での実行が完了しました。  

引き続きSpring Bootを見ていきますが、本記事は一旦ここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>


