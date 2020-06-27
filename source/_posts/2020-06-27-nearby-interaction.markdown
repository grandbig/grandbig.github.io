---
layout: post
title: "iOS14からの新機能：NearbyInteraction Framework"
date: 2020-06-27 23:39
comments: true
categories: ios ios14 swift
---

### はじめに
先日の[WWDC2020](https://developer.apple.com/wwdc20/)で `NearbyInteraction Framework` というものが発表されました。  
これは複数端末間の距離や方角を計測するために利用できる `Framework` だそうです。  

今年はコロナの影響で、ソーシャルディスタンスという言葉が叫ばれるようになったこともあり、意識せざるを得ないような内容に感じられたため、  
WWDC2020で行われた『[Meet Nearby Interaction](https://developer.apple.com/videos/play/wwdc2020/10668/)』セッション動画を元に勉強していきたいと思います。

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### NearbyInteraction Frameworkの機能を利用できる端末
iPhone11以降でU1チップを備えている端末に限られます。  

現段階では、  

* iPhone11  
* iPhone11 Pro  
* iPhone11 Pro Max  

の3端末に限られています。  

※シミュレータが対応しているため、上記実機が手元になくても試すことができます。  

### NearbyInteraction Frameworkで測定できるもの
まず、 `NearbyInteraction Framework` で測定可能なものですが、  

* 距離  
* 方角   

の2つになっています。  

上記2つは [NINearbyObject](https://developer.apple.com/documentation/nearbyinteraction/ninearbyobject) に格納された状態で取得します。  
それぞれ  

```objective-c
@available(iOS 14.0, *)
extension NINearbyObject {

    public var distance: Float? { get }

    public var direction: simd_float3? { get }
}
```

と定義されています。  

### NearbyInteraction Frameworkの利用方法
ここからは詳細について見ていきましょう。  

#### discoveryTokenとは
端末間の距離や方角を計測するとなると、互いの端末を検知し、ペアリングする必要がありそうな気がします。  
これを可能にするために、 `NearbyInteraction Framework` では `NISession` オブジェクトを生成し、  
その `NISession` が提供する `discoveryToken` を利用する必要があります。  

下記のように `discoveryToken` が `NISession` 内に定義されています。  

```objective-c
@available(iOS 14.0, *)
open class NISession : NSObject {
  ....
  @NSCopying open var discoveryToken: NIDiscoveryToken? { get }
}
```

因みに、この `discoveryToken` は  

* ランダムな識別番号として生成される  
* `session` と同等の有効期限が存在する  
* `session` の無効にしたり、アプリを終了すると `discoveryToken` も無効になる  

という性質があります。  

この `discoveryToken` を端末間で共有し合うことで、距離や方角を計測する端末を認識することができるわけです。  

#### discoveryTokenの共有方法
では、 `discoveryToken` の端末間での共有方法は何があるかというと、  
WWDC2020のセッションでは、 `Multiple Connectivity Framework` が紹介されていました。  

また、 `discoveryToken` は `NIDiscoveryToken` 型であり、  

```objective-c
@available(iOS 14.0, *)
open class NIDiscoveryToken : NSObject, NSCopying, NSSecureCoding {
}
```

のように `NSSecureCoding` に準拠しているため、下記のように簡単にエンコードすることができます。  

```objective-c
let session = NISession()
session.delegate = self

let token = session.discoveryToken
let encodedData = try? NSKeyedArchiver.archivedData(withRootObject: token, requiringSecureCoding: true)
```

`Multiple Connectivity Framework` を利用する場合は、下記のように送信します。  

```objective-c
private var mcSession: MCSession!

do {
    try mcSession.send(data, toPeers: mcSession.connectedPeers, with: .reliable)
} catch let error {
    NSLog("Error sending data: \(error)")
}
```

逆に他端末から送信されたエンコードされたデータを受信したら、  

```objective-c
guard let discoveryToken = try? NSKeyedUnarchiver.unarchivedObject(ofClass: NIDiscoveryToken.self, from: data) else {
    fatalError("Unexpectedly failed to encode discovery token.")
}
```

のようにデコードし、 `discoveryToken` を取り出します。  
これで互いに計測対象端末の `discoveryToken` が共有できました。  

#### 計測値の取得方法
あとは、受信した `discoveryToken` を元に生成した `config` を利用してセッションを開始します。  

```objective-c
let config = NINearbyPeerConfiguration(peerToken: discoveryToken)
session?.run(config)
```

対象端末との距離や方角の値は `NISessionDelegate` を通じて取得します。  
下記の `didUpdate` メソッドにて、最新の `NINearbyObject` 情報が取得できます。

```objective-c
@available(iOS 14.0, *)
public protocol NISessionDelegate : NSObjectProtocol {
  // This is called when new updates about nearby objects are available.
  optional func session(_ session: NISession, didUpdate nearbyObjects: [NINearbyObject])
  ...
}
```

`NISessionDelegate` には他にも幾つかメソッドが用意されています。  

例えば、 `didRemove` メソッドは、  

```objective-c
@available(iOS 14.0, *)
public protocol NISessionDelegate : NSObjectProtocol {
  ...
  // This is called when the system is no longer attempting to interact with some nearby objects.
  optional func session(_ session: NISession, didRemove nearbyObjects: [NINearbyObject], reason: NINearbyObject.RemovalReason)
  ...
}
```

ペアリングしたはずの計測対象端末から情報が取得できない場合に呼び出されます。  
これが呼び出されるパターンは主に次の2つになります。  

* 計測対象端末が遠く離れてしまい、情報取得可能な期間をタイムアウトした場合  
* 計測対象端末がセッションを終了した場合  

上記理由も、メソッド引数の `reason` に格納されているため、識別することができます。  

```objective-c
extension NINearbyObject {

    @available(iOS 14.0, *)
    public enum RemovalReason : Int {
        case timeout = 0
        case peerEnded = 1
    }
}
```

また、アプリの状態に関わるメソッドも用意されています。  

```objective-c
@available(iOS 14.0, *)
public protocol NISessionDelegate : NSObjectProtocol {
    ...
    // This is called when a session is suspended.
    optional func sessionWasSuspended(_ session: NISession)

    // This is called when a session may be resumed.
    optional func sessionSuspensionEnded(_ session: NISession)
    ...
}
```

`sessionWasSuspended` はアプリがFG起動を終了し、サスペンド状態になった際に呼び出され、  
`sessionSuspensionEnded` はアプリが再びFG起動に戻し、サスペンド状態が終了した際に呼び出されます。  

因みに、セッションは一度停止すると自動的に再開はされないため、 `sessionSuspensionEnded` 内で再度セッションを開始する必要があります。  

```objective-c
func sessionSuspensionEnded(_ session: NISession) {
    if let config = self.session?.configuration {
        session.run(config)
    }
}
```

最後の `Delegate` メソッドとして `didInvalidateWith` が用意されています。  
これはセッションが無効になった場合に呼び出されます。   

```objective-c
@available(iOS 14.0, *)
public protocol NISessionDelegate : NSObjectProtocol {
    ...
    // This is called when a session is invalidated.
    optional func session(_ session: NISession, didInvalidateWith error: Error)
}
```

セッションが無効になると、 `run` メソッドを使っても再開することができないため、  
セッションの生成からやり直す必要があります。  
(セッションを生成し、 `discoveryToken` を交換し、セッションを開始する一連の流れを頭からやり直します。 )  

### NearbyInteraction Frameworkで計測する上での制約
`NearbyInteraction Framework` は非常に強力な計測ができますが、  
その精度を高めるためには幾つかユーザに制約を課さざるを得ないようです。  

その制約とは、  

1. 方角の測定可能フィールド上に両端末が存在していること  
2. 両端末ともに `Portrait` の向きで端末を持っていること  
3. 両端末間に壁や人などの障害物がないこと  

が上げられています。  

以下、セッションの資料がわかりやすかったため、引用させて頂きます。  

**方角の測定可能フィールド上に両端末が存在していること**  

![方角の測定可能フィールド上に両端末が存在していること](/images/nearbyinteraction_1.png)  

**両端末ともにPortraitの向きで端末を持っていること**  
![両端末ともにPortraitの向きで端末を持っていること](/images/nearbyinteraction_2.png)  

**両端末間に壁や人などの障害物がないこと**  
![両端末間に壁や人などの障害物がないこと](/images/nearbyinteraction_3.png)  

### まとめ
さて、如何でしたでしょうか。  
利用上の制約があることで万能ではないと感じる方もいるかもしれませんが、  
cm単位の精度で端末間の距離や方角がわかるという機能は、  
これからの新生活で大活躍する可能性も秘めているかもしれません。  

個人的には、  
何か世の中の役に立つようなサービスを考えるきっかけになったかなと思いました。  

最後に、今回説明させて頂いた内容は、冒頭に書きました通り[Meet Nearby Interaction](https://developer.apple.com/videos/play/wwdc2020/10668/)動画で確認できますし、  
[Implementing Interactions Between Users in Close Proximit](https://developer.apple.com/documentation/nearbyinteraction/implementing_interactions_between_users_in_close_proximity)にてサンプルコードも公開されています。  

より詳細を見る場合は当然ですが、上記を見ることをオススメいたします。  

といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
