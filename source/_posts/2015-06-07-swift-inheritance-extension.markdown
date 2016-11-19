---
layout: post
title: "Swiftで継承とカテゴリってどうやってやるの！？"
date: 2015-06-07 23:30
comments: true
categories: ios swift
---

####Swiftで継承
まずはObjective-CとSwiftの継承の書き方の違いを見て行きましょう。  

早速、Objective-Cの継承ですが、  

```objective-c
// ヘッダーファイル
@interface SampleA:NSObject
// プロパティの定義
@property(strong, nonatomic) NSString *sampleString;
// メソッドの定義
- (void)sampleMethod;

@end

```

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

```objective-c
// モデルファイル
@interface SampleA()

@end

@implementation

- (void)sampleMethod {
	....
}

@end

```

といった形で書けます。  

これがSwiftだと、  

```objective-c
// swiftファイル
class SampleA:NSObject {
	// プロパティの定義
	var sampleString:String = "サンプル"
	// メソッドの定義
	func sampleMethod {
		....
	}
}

```

とかなりシンプルに書けますね！  

####Swiftでカテゴリ
次にSwiftでObjective-Cで言うカテゴリを実装してみます。  

Objective-Cでは、  

```objective-c
// ヘッダーファイル
@interface NSError(ContextBase)

- (NSError *)createError:(NSString *)message code:(NSInteger)code;

@end
```

```objective-c
// モデルファイル
@implementation NSError(ContextBase)

 - (NSError *)createError:(NSString *)message code:(NSInteger)code {
	NSDictionary *errorUserInfo = @{ NSLocalizedDescriptionKey:message };
	NSError *error = [[NSError alloc] initWithDomain:domain code:code userInfo:errorUserInfo];

	return error;
}

@end
```

と書いてきました。  
これがSwiftでは、  

```objective-c
// Swiftファイル
extension NSError {
	func createError(message:String!, code:Int!) -> NSError {
		let errorUserInfo = [NSLocalizedDescriptionKey: message]
		let error:NSError = NSError(domain: "jp.co.swiftSample", code: code, userInfo: errorUserInfo)

		return error
	}
}
```

と書けます。  
う〜んこれもシンプル！！  
Objective-Cで利用してきた継承はもちろんのこと、カテゴリも残っていて助かりますね！  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
