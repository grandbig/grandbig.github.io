---
layout: post
title: "Spring Bootでリダイレクト先にパラメータを渡す方法"
date: 2016-05-28 20:39
comments: true
categories: java springboot
---

####RedirectAttributesを使ってみよう！
さて、本日は久々にSpring Bootの話です。  

皆さんはWebアプリケーションを作る中で、「リダイレクト先にパラメータを渡したい！」なんてことはありませんでしょうか？  
筆者は先日、そのような状況に鉢合わせたのですが、ググってみてもなかなか求めている答えが見つからず、少々悩んでしまいました。  
GETリクエストであればリダイレクト後にURLからリクエストパラメータが取得できますが、  
POSTリクエストやURLにリクエストパラメータを表示するわけにいかない仕様の場合はそうもいきません。  

そんなときに役に立つのが`RedirectAttributes`です。  
では、早速、使い方を見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
<!-- more -->

####リダイレクト先に文字列を送る
まずは単純な例から見ていきましょう。  
下記はリダイレクト先に文字列をパラメータとして渡す例です。    

```java
// HelloController
@Controller
public class HelloController {
	@RequestMappinf(value = "/", method = RequestMethod.POST)
	public String form(RedirectAttributes redirectAttributes, @RequestParam("name") String name) {
		
		redirectAttributes.addFlashSttribute("name", name);
		return "redirect:/redirectSample";
	}

	@RequestMapping("/redirectSample")
	public String sample(@ModelAttributes("name") String name) {
		
		System.out.println("name: " + name);
		return "sample";
	}
}
```

実際に動かしてみれば、正しくログが出力されることを確認できるでしょう。  
上記で`return "sample";`としているのは単なる例なので、適切なtemplateを指定してあげてください。  

####リダイレクト先にオブジェクトを送る
続いて、パラメータをまとめて送るパターンです。  
今回は下記のように定義された`MyData`を送ってみましょう。  
※ソースを簡単にするために`build.gradle`に`compile("org.projectlombok:lombok:1.16.6")`を設定して`@Data`を利用しています。  

```java
// MyData.java
@Data
public class MyData {

	private String name;
	private String memo;
}
```

では、実際にパラメータを送ってみましょう。  

```java
// HelloController

@Controller
public class HelloController {

	@RequestMapping(value = "/", method = RequestMethod.POST)
	public String form(RedirectAttributes redirectAttributes, @RequestParam("name") String name, @RequestParam("memo") String memo) {
		
		// MyDataについては後述します
		MyData mydata = new MyData(name, memo);
		ModelMap modelMap = new ModelMap();
		modelMap.addAttribute("mydata", mydata);
		redirectAttributes.addFlashAttribute("model", modelMap);

		return "redirect:/redirectSample";
	}

	@RequestMapping("/redirectSample")
	public String sample(@ModelAttribute("model")ModelMap modelMap) {
		
		MyData mydata = modelMap.get("mydata");
		String name = mydata.getName();
		String memo = mydata.getMemo();
		System.out.println("name: " + name + ", memo: " + memo);

		return "sample";
	}
}
```

オブジェクトとしてパラメータを送るには`ModelMap`に一度格納する必要があります。  
パラメータを受け取ったあとに型変換して取り出せばOKです。  

####RedirectAttributesにおけるaddAttributeとaddFlashAttributeの違い
さて、今回は`RedirectAttributes`の`addFlashAttribute`を利用しましたが、`RedirectAttributes`には`addAttribute`というメソッドも存在します。  
ではなぜ`addFlashAttribute`を利用したのか参考までに書いておきます。  

#####RedirectAttributesのaddAttributeはリダイレクト先に文字列としてパラメータを送る
大きな違いは`addFlashAttribute`のように`ModelMap`型でパラメータを送れません。  
よって、複数のパラメータを送るには1つずつパラメータをセットするしか方法がありません。  

```java
// addAttributeの例
redirectAttributes.addAttribute("name", "hogehoge");
redirectAttributes.addAttribute("memo", "fugafuga");
```

#####RedirectAttributesのaddAttributeはリダイレクト先でのパラメータの受取り方が異なる
こちらも大きな違いとなりますが、`addFlashAttribute`のときのように`@ModelAttribute`を使ってパラメータを受け取ることができません。  
`addAttribute`の場合、`@RequestParameter`を使ってパラメータを受取ります。  

```java
// addAttributeの例
@RequestMapping("/redirectSample")
public String sample(@RequestParam("name") String name, @RequestParam("memo") String memo) {
	...
}
```

#####リロード時の挙動が異なる
最後になりますが、  
`addAttribute`と`addFlashAttribute`ではリダイレクト後にリロードすると挙動が異なります。  
`addAttribute`ではGETリクエストとしてURLの末尾に`?name=hogehoge&memo=fugafuga`という形でパラメータが渡されるため、リロードをしてもパラメータを受け取ることができます。  

しかし、`addFlashAttribute`では`flashMap`を利用したパラメータ渡しとなっています。  
よって、リダイレクト後は`flashMap`からパラメータが残らないため、何もパラメータを受け取ることはできなくなります。  


いかがでしたでしょうか？  
筆者にとってもSpring Bootに慣れない日々が続きますが、気づいたことや学んだことについてはブログにまとめていきたと思います。  
と言ったところで本日はここまで。  


<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

