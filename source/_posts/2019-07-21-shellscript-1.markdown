---
layout: post
title: "シェルスクリプトで特定フォルダ配下のファイル名をチェックしよう！"
date: 2019-07-21 22:52
comments: true
categories: shellscript
---

### はじめに
大量のファイルを新規作成して、そのファイルが一定のルールを持った名称であるかどうかをチェックする際に、流石に目視で確認するのは大変過ぎるので、  
シェルスクリプトで確認できるようにしてみました。

その時のメモとして残しておこうと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### 実装したシェルスクリプトの説明
例として、今回は、iOSアプリ内で利用する画像ファイル名に `@2x` , `@3x` が付与されていることを確認するためのシェルスクリプトを作りました。   

実際の処理内容は下記の通りで、ルールにそぐわないファイルを出力してくれます。  

```sh
# check_filename.sh
#!/bin/sh

# 引数として指定されたフォルダ配下を見る
folderPath=$1

echo "問題のあるファイルは..."

for filepath in $folderPath; do

  # 〜@2x.png または 〜@3x.pngのファイル以外を正規表現で抽出する
  if [[ ! $filepath =~ .+@[23]x.png ]]; then
    echo $filepath
  fi
done

echo "です。"
```

実行方法は下記の通りです。  

相対パスの場合は、  

```sh
$ ./check_filename.sh "./*"
```

絶対パスの場合は、  

```sh
$ ./check_filename.sh "/Users/xxxxxxx/Desktop/test_dir/*"
```

のように指定できます。  

### まとめ
極力、ミスの起こりうる手動ではなく、自動で効率的に業務を遂行していきたいですね。  
と言うことで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
