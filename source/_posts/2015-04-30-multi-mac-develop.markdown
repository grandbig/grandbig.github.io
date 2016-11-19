---
layout: post
title: "複数のMacでiOS端末の実機検証を可能にするためには！？"
date: 2015-04-30 18:55
comments: true
categories: mac
---

####証明書と鍵を書きだそう！
本日は複数のMacでiOS端末の実機検証をするための手順について説明します。  

まず、開発者用の証明書を書き出しましょう。  
証明書はキーチェーンアクセスから確認できます。  
キーチェーンアクセスアプリを起動すると、下記画面が表示されます。  

![キーチェーンアクセスの画面](/images/multi_mac_1.png)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

まずは証明書を書き出しましょう。  
書き出したい証明書を右クリックで『iPhone Developer: ☓☓☓☓を書き出す』を選択しましょう。  

![証明書の書き出し](/images/multi_mac_2.png)  

名前をつけて保存する際に、パスワードの入力を求められます。  
これは他のMacで証明書を読み込むときに必要となるので、きっちり決めておきましょう。  

![パスワードを設定](/images/multi_mac_3.png)  

次に鍵を書き出しましょう。  
先ほど書き出した証明書をプルダウンして、鍵を表示しましょう。  
書き出したい鍵を右クリックで『"☓☓☓☓"を書き出す』を選択しましょう。  

![鍵の書き出し](/images/multi_mac_4.png)  

こちらも同様にパスワードを設定します。  
![パスワードを設定](/images/multi_mac_5.png)  

以上で書き出しが終了しました。  
続いて、他のMacで証明書と鍵を読み込んでみましょう。  

####証明書と鍵を読み込もう
先ほど書き出した証明書と鍵を他のMacに持ってきます。  
証明書から先にダブルクリックで読み込みましょう。  

すると下図のようにパスワード入力画面が要求されるので、先ほど自身で設定したパスワードを入力しましょう。  

![パスワード要求画面](/images/multi_mac_6.png)  

続いて、鍵をダブルクリックして読み込みましょう。  
同様に下図のようなパスワード入力画面が要求されます。自身で設定したパスワードを入力しましょう。  

![パスワード要求画面](/images/multi_mac_7.png)  

これで証明書と鍵の読込みは完了です。  
キーチェーンアクセスを開いて、書き出した方のMacと同じ証明書と鍵があることを確認しましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
