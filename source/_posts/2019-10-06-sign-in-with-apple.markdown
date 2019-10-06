---
layout: post
title: "iOS13から対応する『Sign In with Apple』〜サードパーティアカウントでOAuth認証によるログインを備えたアプリで迷わないように。〜"
date: 2019-10-06 17:05
comments: true
categories: ios ios13
---

### はじめに
今回はiOS13から導入された `Sign In with Apple` について見ていきたいと思います。  

もし、 `Facebook / Google / Twitter` などのアカウントを用いたログインを可能とする機能をアプリが持っている場合、  
Appleが新たに提唱した `Sign In with Apple` の機能を実装する必要性が出てきたようです。  

一方で `Facebook / Google / Twitter` などの本家のアプリでは恐らく実装する必要はないと思われます。  
(ログイン機能を実装するのであれば、 `Sign In with Apple` が必須という話ではなく、サードパーティ製のログイン機能を持つアプリに限るようです。 )    

とは言え、「急にそんなことを言われても対応工数がかかるし、他に実装したい機能もあるし...」と困るエンジニアもいるかもしれません。  

ですが、 `Apple` が要求する以上、 iOSアプリを開発し続ける上で避けては通れない問題ですので、簡単に対応方法を紹介したいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### Apple Developer Programでの対応
まずは、コードを書く前に、 `Sign In with Apple` の機能を有効にした `App ID` を作成し、 `Provisioning Profile` に紐付ける必要があります。  

#### App IDの作成
下記手順で `Sign In with Apple` 対応の `App ID` を作成します。  

まずは、 [Apple Developer Program > Certificates, Identifiers & Profiles > Identifiers](https://developer.apple.com/account/resources/identifiers/list) にアクセスします。  

追加するための「＋」ボタンを選択後に表示される下記の画面で `App IDs` を選択します。  
![App IDsを選択](/images/sign-in-with-apple_1.png)  

必要事項を入力します。  
![DescriptionとBundle IDを入力](/images/sign-in-with-apple_2.png)  

入力例)  

* Description: `SignInWithAppleSample`  
* Bundle ID: `com.xxx.SignInWithAppleSample`  

下にスクロールすると `Sign In with Apple` の項目が追加されているので、チェックをつけます。  
![Sign In with Appleにチェック](/images/sign-in-with-apple_3.png)   

上記状態で次に進み、確認画面で問題ないことを確認すれば、 `Register` ボタンを選択するだけで作成完了です。  
![確認完了後、Registerボタンを選択](/images/sign-in-with-apple_4.png)  

#### Provisioning Profileへの紐付け
下記手順で `App ID` を紐付けて `Provisioning Profile` を作成します。  

[Apple Developer Program > Certificates, Identifiers & Profiles > Profiles](https://developer.apple.com/account/resources/profiles/list)にアクセスします。  

追加するための「＋」ボタンを選択後に表示される下記の画面で `iOS App Development` を選択します。  
(開発時を想定しているため、リリース時であれば、 `App Store` など適切なものを選択してください。)  
![iOS App Developmentを選択](/images/sign-in-with-apple_5.png)  

先程作成した `App ID` を選択します。  
![App IDを選択](/images/sign-in-with-apple_6.png)  

必要な `Certificate` を選択します。  
![Certificateを選択](/images/sign-in-with-apple_7.png)  

利用したいデバイスを選択します。  
![Deviceを選択](/images/sign-in-with-apple_8.png)  

`Provisioning Profile` の名前を決めます。  
![Provisioning Profileの名前を入力](/images/sign-in-with-apple_9.png)  

`Generate` ボタンを選択すれば作成完了です。  

### Xcode11の設定
`Apple Developer Program` 上での準備が整ったら、 `Xcode11` で `Sign In with Apple` の `Capability` を追加しましょう。  

プロジェクトを選択し、 `Signing & Capabilities` タブから追加します。  
![Sign In with AppleのCapabilityを追加](/images/sign-in-with-apple_10.png)   

### ソースコードの実装
ソースコードを書いていきましょう。  
これは `Apple` が公開しているサンプルを一部抜粋して説明します。  
サンプルは[こちら](https://developer.apple.com/documentation/authenticationservices/adding_the_sign_in_with_apple_flow_to_your_app)からダウンロードできます。  

`Sign In with Apple` は `AuthenticationServices` の各種メソッドを利用するため、  
まずは `AuthenticationServices` をインポートします。  

```objective-c
import UIKit
import AuthenticationServices

class LoginViewController: UIViewController {
    ...
}
```

続いて、ログイン画面に `Sign In with Apple` のボタンを配置します。  

```objective-c
class LoginViewController: UIViewController {

    /// Sign In with Appleボタンを追加するためのUIStackView
    @IBOutlet weak var loginProviderStackView: UIStackView!

    override func viewDidLoad() {
        super.viewDidLoad()

        // Sign In with Appleボタンを配置します
        setupProviderLoginView()
    }

    ...

    /// Sign In with Appleのボタンを配置
    func setupProviderLoginView() {
        let authorizationButton = ASAuthorizationAppleIDButton()
        authorizationButton.addTarget(self, action: #selector(handleAuthorizationAppleIDButtonPress), for: .touchUpInside)
        self.loginProviderStackView.addArrangedSubview(authorizationButton)
    }

    ...

    /// Sign In with Appleボタンタップ後のアクション
    @objc
    func handleAuthorizationAppleIDButtonPress() {
        let appleIDProvider = ASAuthorizationAppleIDProvider()
        let request = appleIDProvider.createRequest()
        request.requestedScopes = [.fullName, .email]

        let authorizationController = ASAuthorizationController(authorizationRequests: [request])
        authorizationController.delegate = self
        authorizationController.presentationContextProvider = self
        authorizationController.performRequests()
    }
}
```

上記を見てわかるように、タップ時に呼び出すメソッドを定義して、その中で必要なリクエストを生成し、実行しています。  

`requestedScopes` は必要なアクセス情報の範囲を定義するようですが、  
今のところは `.fullName` と `.email` しか用意されていないようです。  

また、 `delegate` と `presentationContextProvider` を呼び出す必要があります。  
これは、

* `ASAuthorizationControllerDelegate` の `authorizationController:didCompleteWithAuthorization:` で認証完了をキャッチする  
* `ASAuthorizationControllerPresentationContextProviding` の `presentationAnchorForAuthorizationController:` で認証画面に必要な画面の提示場所を設定する  

ためです。  

```objective-c
extension LoginViewController: ASAuthorizationControllerDelegate {
    func authorizationController(controller: ASAuthorizationController, didCompleteWithAuthorization authorization: ASAuthorization) {
        ...
    }
}

extension LoginViewController: ASAuthorizationControllerPresentationContextProviding {
    func presentationAnchor(for controller: ASAuthorizationController) -> ASPresentationAnchor {
        return self.view.window!
    }
}
```

### 動作確認
ここまで実装した内容で、どんな動きになるのかを見てみます。

#### iPhone上でAppleIDでサインインしていない場合
あまりケースとして多くはないかもしれませんが、普通にありえるので実装時に考慮が必要です。  
この場合、 `performRequests()` を実行した時点で『設定アプリでAppleIDアカウントでサインインするかどうか』を聞かれます。  

![設定アプリでAppleIDアカウントでサインインするかどうか聞かれます](/images/sign-in-with-apple_11.png)   

サインイン後に、改めてアプリに戻りログインを実行することで先に進めるようになります。  

#### iPhone上でAppleIDでサインイン済みの場合
改めて `Sign In with Apple` のボタンをタップすると、以下の画面が表示されます。  

![Sign In with Appleの画面](/images/sign-in-with-apple_12.png)   

`Continue` ボタンのタップで先に進むと、以下の画面が表示されます。  

![Share My Email / Hide My Email選択画面](/images/sign-in-with-apple_13.png)  

ここで `Share My Email` または `Hide My Email` のどちらかを選択することになります。  
`Hide My Email` はよりユーザにとってセキュアな状態を保つために用意されており、以下の特徴があります。  

* 実際のメールアドレスとは異なるメールアドレスが発行される  
* そのメールアドレスは、認証時に利用したAppleIDのメールアドレスとリンクしている  
* アプリに実際のメールアドレスを渡さないで済む  

上記選択した上で `Continuer with Password` ボタンをタップして次に進むと、以下の画面が表示されます。  

![パスワード入力画面](/images/sign-in-with-apple_14.png)  

パスワード入力して次に進むと、二段階認証を求められます。  

![二段階認証画面](/images/sign-in-with-apple_15.png)  

二段階認証してやっと認証が完了となります。  

### まとめ
さて如何でしたでしょうか。  
アプリの機能次第で対応必須な内容ですので、今からでも知っておいて損はないかなと思います。  
`SDK` 自体は非常に簡単に扱えますので、アプリ側の実装は早めに目処をつけておいて、データ構成など課題がある部分にフォーカスした方が良いでしょう。  

と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
