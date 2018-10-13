---
layout: post
title: "HackerRankで問題を解いてみよう！"
date: 2018-07-22 23:35
comments: true
categories: ios swift
---

### はじめに
さて、今回は[HackerRank](https://www.hackerrank.com/dashboard?h_r=logo)について紹介したいと思います。  
HackerRankとは求人サイトの一種で、問題を解くことで得点を稼ぐことができ、そのスコアでランキングを競うことのできるサイトです。  
同様のサービスでは[paiza](https://paiza.jp/)も有名かと思います。  
( 少し前はCodeIQといったサービスもありましたね。 )  

HackerRankの特徴は、  

* 全世界のエンジニア対象(=ランキングは世界ランキングになる)  
* 問題文は全て英語で書かれている  
* 問題種別は `Algorithms`, `Data Structures`, `Mathematics` が用意されている  

といったところでしょうか。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### どんな問題なのか
オンラインでのプログラミングテストでありがちなのが、その独特な `Input` の扱い方かと思います。  
例えば、複数 `Input` がある場合、  
1回目の `readLine()` の呼び出しが1つ目の `Input` に当たり、  
2回目の `readLine()` の呼び出しが2つ目の `Input` に当たるといった感じですね。  

HackerRankでも、初級の問題ではそういった部分を書かせることがあるのですが、  
ほとんどの問題が『肝心のロジックをどのように書いているのかに終始できる問題の出し方になっています。  

では、実際に一例を見てみましょう。  
こちらの[Apple and Orange](https://www.hackerrank.com/challenges/apple-and-orange/problem)は `Algorithms` に区分けされた `Easy` 問題です。  

問題を見てみると、「何やら既にたくさんのコードが書かれているな」ということに気づくでしょう。  

![HackerRankの問題例](/images/hacker_rank.png)  

上図のように中身のないメソッドが用意されており、その中のロジックを書くことが『問題に回答する』ということになります。  
`Easy` 問題であっても、割と長々と英語で問題が解説されているため、身構えてしまう人もいるかもしれませんが、  
最後まで読み切ってしまえば、問題の難易度と英文の長さが比例しているわけではないということがわかるでしょう。  

参考までに上記の問題を筆者が解いたときの回答を記載しておきます。  

```objective-c
func countApplesAndOranges(s: Int, t: Int, a: Int, b: Int, apples: [Int], oranges: [Int]) -> Void {
    // まずは林檎の計算から開始する
    var sumAppleCount = 0
    for apple in apples {
        let position = a + apple
        if position >= s, position <= t {
            sumAppleCount += 1
        }
    }

    // 次にオレンジの計算
    var sumOrangeCount = 0
    for orange in oranges {
        let position = b + orange
        if position >= s, position <= t {
            sumOrangeCount += 1
        }
    }

    print("\(sumAppleCount)\n\(sumOrangeCount)")
}
```

このように、コメント文を日本語で書いても問題はありません。  
因みに、一度解いて提出した回答内容はいつでも `Submissions` タブから確認することができます。  

![提出した回答の確認方法](/images/hacker_rank_2.png)  

### まとめ
さて、今回はHackerRankについて簡単に紹介しました。  
初めは『プロダクトの開発の方がよっぽど面白いんじゃないか』と思っていたのですが、  
案外、解き始めて見ると楽しいし、勉強になるしで良い遊び場を見つけたような気持ちになりました。  

少しずつではありますが、筆者も `Easy` 問題から解き始めています。  
そして [GitHub](https://github.com/grandbig/HackerRankAnswer)に、その軌跡を残し始めることにしました。  

「プロダクト開発で、『○○なロジックをキレイに書くためには...』なんて考えた時に思い出すかもしれないな」なんて思ったり笑  

と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
