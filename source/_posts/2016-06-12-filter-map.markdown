---
layout: post
title: "SwiftとJava8とJavaScriptでreduce / filter / map / flatMap"
date: 2016-06-12 21:01
comments: true
categories: ios swift java javascript
---

####SwiftとJava8とJavaScriptを比較してみよう！
本日は前々から気になっていた『SwiftとJava8とJavaScript』の比較をしてみたいと思います。  
と言っても難しいことをやるわけではなく、今回はreduce, filter, map, flatMapメソッドの書き方を比較してみます。  

筆者個人としては、どうしてもSwiftやJava8から書き方やメソッドの意味の理解を始めようとすると時間がかかってしまいます。  
なので、JavaScriptから入って比較することで理解が促進することがあるのです。  

では早速見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####reduceメソッド
まずは`reduce`メソッドです。

#####JavaScript

```javascript
var array = [1, 2, 3, 4, 5];
var reduced = array.reduce(function(previus, current) {
	return previous + current;
}, 0);

// 結果 -> 15
```

#####Swift

```objective-c
let array = [1, 2, 3, 4, 5]
let reduced = array.reduce(0) { (previous, current) -> Int in
	previous + current
}

// 結果 -> 15
```

#####Java8

```java
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);
Optional<Integer> sum = list.stream().reduce((previous, current) -> previous + current);

// 結果 -> 15
```

####filterメソッド
次に`filter`メソッドです。  

#####JavaScript

```javascript
var array = [1, 50, 800, 3, 44];
var filtered = array.filter(function(elem) {
	return elem >= 10;
});

// 結果 -> [50, 800, 44]
```

#####Swift

```objective-c
let array = [1, 50, 800, 3, 44]
var filtered = array.filter { (elem) -> Bool in
	elem >= 10
}

// 結果 -> [50, 800, 44]
```

#####Java8

```java
List<Integer> list = Arrays.asList(1, 50, 800, 3, 44);
List<Integer> filteredList = new ArrayList<Integer>();
list.stream().filter(elem -> elem >= 10).forEach(elem -> filteredList.add(elem));

// 結果 -> [50, 800, 44]
```

####mapメソッド
続いて`map`メソッドです。  

#####JavaScript

```javascript
var array = [1, 2, 3, 4, 5];
var mapped = array.map(function(elem) {
	return elem * elem;
});

// 結果 -> [1, 4, 9, 16, 25]
```

#####Swift

```objective-c
let array = [1, 2, 3, 4, 5]
var mapped = array.map { (elem) -> Int in
	elem * elem
}

// 結果 -> [1, 4, 9, 16, 25]
```

#####Java8

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> mappedList = new ArrayList<Integer>();
list.stream().map(elem -> elem * elem).forEach(elem -> mappedList.add(elem));

// 結果 -> [1, 4, 9, 16, 25]
```

####flatMapメソッド
最後に`flatMap`メソッドです。  
(と言いつつ、処理の意味的には`flatten`しか入っていませんね...)  

#####JavaScript
JavaScriptでは標準で`flatMap`メソッドは実装されていません。  
自作するしかないわけですが、`flatMap = flatten + map`なので下記のように書けます。  

```javascript
var listArrayList = [[1, 2], [3], [4, 5]]
var flatMappedList = Array.prototype.concat.apply([], listArrayList).map(function(elem) { 
	return elem;
});

// 結果 -> [1, 2, 3, 4, 5]
```

#####Swift

```objective-c
var listArrayList:[Int]] = []
let list1: [Int] = [1, 2]
let list2: [Int] = [3]
let list3: [Int] = [4, 5]
listArrayList.append(list1)
listArrayList.append(list2)
listArrayList.append(list3)
												        
let flatMappedList = listArrayList.flatMap { (elem) -> [Int] in
	return elem
}

// 結果 -> [1, 2, 3, 4, 5]
```

#####Java8

```java
List<List<Integer>> listArrayList = new ArrayList<List<Integer>>();
List<Integer> list1 = Arrays.asList(1, 2);
List<Integer> list2 = Arrays.asList(3);
List<Integer> list3 = Arrays.asList(4, 5);
listArrayList.add(list1);
listArrayList.add(list2);
listArrayList.add(list3);
List<Integer> flatMappedList = new ArrayList<Integer>();
listArrayList.stream().flatMap(elem -> elem.stream()).forEach(elem -> flatMappedList.add(elem));

// 結果 -> [1, 2, 3, 4, 5]
```

さていかがでしたでしょうか？  
1つの言語を極めれば、何となく他の言語でも書き方がわかるという話を聞いたりすると思うのですが、こういったことの延長戦にある話なんだろうなと思ったりました。  
と言ったところで本日はここまで。  


<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

