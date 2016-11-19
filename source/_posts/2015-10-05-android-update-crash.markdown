---
layout: post
title: "Android5.0から5.1にアップデートして文鎮化した事件簿"
date: 2015-10-05 22:38
comments: true
categories: android
---

####Android OSのアップデートは恐ろしい
さて、本日はAndroid OSのアップデートに関するお話です。  
筆者は最近、Nexus6をAndroid5.0から5.1にアップデートしようとしたのですが、途中で失敗して端末が文鎮化するという悲劇が発生してしまったので、備忘録です。  

#####文鎮化までの軌跡
まずは、如何にしてNexus6が文鎮化したかまで説明します。  

1. 端末の通知センターからバージョンアップのためのデータをダウンロード
2. 端末を再起動して、インストールしますか？と聞かれるのでYESを選択
3. アップデートに失敗しました。という文言と共に下図が...

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

![エラー発生](/images/android-update.png)  

4. 嘘だろ...と思って端末の電源長押しで再起動
永久、再起動ループに...
5. 電源 + 音量Downボタンを長押しでReboot起動
すると下図のような感じで起動します。  
START, Recovery, Factory...などのメニューから仕掛けたい動作を選択できます。  
![Reboot起動](/images/android-update2.png)
6. 順番に全部やっているけど、どれも無限ループに...

はい！文鎮化しましたね。  

####文鎮化Nexus6を復旧するまでの軌跡
まさか文鎮化したNexus6を放っておくわけにはいきません。  
復旧させましょう！  

1. LOCKを外す
`fastboot oem unlock`を実行して、端末のLOCKを解除しましょう。  
2. [Google Developerサイト](https://developers.google.com/android/nexus/images?hl=ja)からFactory Imageをダウンロード
筆者は5.0.1 (LRX22C)を落としました。  
3. ダウンロードしたフォルダ内にzipファイルがあるので、解凍する
4. 下記のコマンドを実行

```
fastboot flash bootloader bootloader-shamu-moto-apq8084-71.05.img
fastboot reboot-bootloader
sleep 5
fastboot flash radio radio-shamu-d4.0-9625-02.55.04.img
fastboot reboot-bootloader
sleep 5
cd "zipを解凍したフォルダ"
fastboot flash boot boot.img
fastboot flash cache cache.img
fastboot flash recovery recovery.img
fastboot flash system system.img
fastboot flash userdata userdata.img
flashboot reboot
```

5. 再起動したAndroidで初期設定

これで復旧させることができました。やったね!!  

と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
