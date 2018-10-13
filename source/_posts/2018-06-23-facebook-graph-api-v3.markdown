---
layout: post
title: "FacebookのGraph API v3から分離されたuser_genderについて見てみる"
date: 2018-06-23 11:39
comments: true
categories: ios swift facebook
---

### はじめに
最近、いろいろと個人情報の話が取り沙汰されていますが、Facebookが `Graph API v3` を発表しましたね！  
Facebookログインを利用して会員登録の簡略化をするなんて、今や当たり前の時代に、個人情報の保護強化のために情報取得のハードルが少々引き上げられました。  

また、これまでFacebookログインを利用していたサービスはFacebookの再審査を受けることが義務化されました。  
今回新たに審査が必要となる項目も追加され、Facebook社のより一層の個人情報強化の意志を感じさせますね。  
因みに、新規にパーミションが必要となった項目は、  

* `user_link`  
* `user_age_range`  
* `user_gender`  

の3つになります。  
本記事では、中でも `user_gender` に焦点を当てて見ていきたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### user_genderとは
フィールド名から分かる通り、値としては、ユーザの `性別` を取得するものになります。  
これは、新たにパーミッションが必要になった項目ということですが、  
もちろんこれまでも取得できる項目でした。  

どうやって取得していたかと言うと、Facebook上でユーザの基本情報としてまとめられていた `public_profile` に `gender` として含まれていました。  
v3.0での変更でこれが独立したということになります。  

### パーミッション画面での変更
続いて、`public_profile` と `user_gender` を利用した場合でそれぞれユーザに表示するパーミッション画面はどのように変わるのでしょうか？  
実際に見ていきましょう。  

まずは、これまでの `public_profile` を利用した場合の画面は  
![public_profileのみ指定した場合のパーミッション画面](/images/facebook_graph_api_v3_1.png)  

続いて、 `user_gender` を指定した場合の画面は  
![user_genderも指定した場合のパーミッション画面](/images/facebook_graph_api_v3_2.png)  

となります。  
今後どうなるかはわからないものの、今のところは `public_profile` の中に `gender` が含まれており、プラスで `user_gender` も指定できるという状況になっています。  

### パーミッション指定の処理
参考までに今回試したソースコードを記載しておきます。  

```objective-c
// FacebookManager.swift
import Foundation
import PromiseKit
import FacebookCore
import FacebookLogin

public enum Gender: Int {
    case male
    case female
}

public struct FacebookUserProfile: Codable {
    public var id: String
    public var email: String?
    public var gender: Gender

    private enum CodingKeys: String, CodingKey {
        case id
        case email
        case gender
    }

    public init(from decoder: Decoder) throws {
        let values = try decoder.container(keyedBy: CodingKeys.self)
        self.id = try values.decode(String.self, forKey: .id)
        self.email = try values.decodeIfPresent(String.self, forKey: .email)
        let genderString = try values.decode(String.self, forKey: .gender)
        if genderString == "male" {
            self.gender = .male
        } else {
            self.gender = .female
        }
    }

    public func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(id, forKey: .id)
        try container.encodeIfPresent(email, forKey: .email)

        if gender == .male {
            try container.encode("male", forKey: .gender)
        } else {
            try container.encode("female", forKey: .gender)
        }
    }
}

public final class FacebookManager {

    public static let shared = FacebookManager()
    private init() {
    }

    enum FacebookError: Error {
        case cancel
        case apiError
        case decodeError
    }

    public enum FacebookPermission: String {
        case email = "email"
        case publicProfile = "public_profile"
        case userGender = "user_gender"       // 新規パーミッションを指定
    }

    public func application(_ application: UIApplication, didFinishLaunchingWithOptions: [UIApplicationLaunchOptionsKey: Any]?) {

        SDKApplicationDelegate.shared.application(application, didFinishLaunchingWithOptions: didFinishLaunchingWithOptions)
    }

    public func application(_ app: UIApplication,
                            open url: URL,
                            options: [UIApplicationOpenURLOptionsKey: Any] = [:]) -> Bool {

        return SDKApplicationDelegate.shared.application(app, open: url, options: options)
    }

    public func isLoggedIn() -> Bool {
        return AccessToken.current != nil
    }

    public func login(_ viewController: UIViewController,
                      permissions: [FacebookPermission] = [.email, .publicProfile, .userGender]) -> Promise<String> {

        if isLoggedIn() {
            logout()
        }

        let promise = Promise<String> { fulfill, reject in
            var readPermissions = [ReadPermission]()
            permissions.forEach {
                readPermissions.append(ReadPermission.custom($0.rawValue))
            }

            LoginManager().logIn(readPermissions: readPermissions,
                                 viewController: viewController,
                                 completion: { result in

                                    switch result {
                                    case .success(_, _, let accessToken):
                                        _ = fulfill(accessToken.authenticationToken)
                                    case .cancelled:
                                        let error = FacebookError.cancel
                                        _ = reject(error)
                                    case .failed(let error):
                                        _ = reject(error)
                                    }
            })
        }

        return promise
    }

    public func logout() {
        LoginManager().logOut()
    }
}

extension JSONDecoder {
    // Any型をdecode可能なメソッドを追加
    public func decode<T: Decodable>(_ type: T.Type,
                                     withJSONObject object: Any,
                                     options opt: JSONSerialization.WritingOptions = []) throws -> T {

        let data = try JSONSerialization.data(withJSONObject: object, options: opt)
        return try decode(T.self, from: data)
    }
}
```

### まとめ
さて如何でしたでしょうか？  
今後移行期間がどうなるのか、イマイチ見えない状況ではありますが、取り急ぎ8/1までにFacebookの再審査を受けて大事に備えておくべきでしょう。  
といったところで本日はここまで。  

#### 参考

* [Facebook Graph API v3.0](https://developers.facebook.com/docs/graph-api/changelog/version3.0)  
* [FACEBOOK GRAPH API V3.0 リリースと同時にログイン審査がリニューアルされた件](https://damelog.com/sns/facebook/facebook-graph-api-v3_0-released-and-app-review-restored/)

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
