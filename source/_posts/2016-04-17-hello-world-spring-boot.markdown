---
layout: post
title: "Spring BootでHello Worldをしてみよう！"
date: 2016-04-17 01:25
comments: true
categories: springboot java
---

####Spring BootでHello World
引き続きSpring Bootの練習です。  
前回、Spring Boot用のプロジェクトを作成して、起動してみるところまでやってみました。  
今回はHello Worldアプリケーションを作成してみます。  
(Node.jsのExpressとは異なりますが、結構簡単にできました。)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

まずは、下記が前回のフォルダ構成です。  

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

今回はWebアプリケーションを作成するので、`build.gradle`を修正する必要があります。

```java
// build.gradle
dependencies {
	compile 'org.springframework.boot:spring-boot-starter-web'
	testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

コンパイル対象のSpring Bootのモジュールを変更しています。  
(元はcompile('org.springframework.boot:spring-boot-starter')でした。)  

次に、上記のフォルダ構成の`FirstApplication.java`と同階層に`HelloController.java`を作成します。  

```java
// HelloController.java
package com.example;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/hello-world")
public class HelloController {
	@RequestMapping(method=RequestMethod.GET)
	public String hello() {
		return "Hello Spring MVC";
	}
}
```

`HelloController.java`の処理について説明すると  

* `@RestController`: WebAPIのエントリポイントクラスを指定  
* `@RequestMapping`: パスやHTTPメソッド(GET, POST, PUT, DELETEなど)のマッピングを指定  
  * `@RequestMapping("/hello-world")`で`HelloController`クラスにパスをマッピング
  * `@RequestMapping(method=RequestMethod.GET)`で`hello`メソッドにHTTPメソッドをマッピング  

そして、`application.properties`にポート番号を指定します。  

```java
// application.properties
server.port=9000
```
あとは、ターミナルから`gradle bootRun`を実行するだけです。  
そして、Chromeで`http://localhost:9000/hello-world`にアクセスしてみましょう。  
下記のような結果が得られるはずです。  

![Hello World](/images/hello-world-spring-boot1.png)  

もろもろの問題はあるけれど、後回しにして、引き続き勉強を進めます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
