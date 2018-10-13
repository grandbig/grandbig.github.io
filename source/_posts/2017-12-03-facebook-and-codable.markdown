---
layout: post
title: "Facebook APIで試すCodable"
date: 2017-12-03 15:30
comments: true
categories: ios swift swift4 facebook codable
---

### はじめに
こちらは [Swift その2 Advent Calendar 2017](https://qiita.com/advent-calendar/2017/swift2) の3日目の記事です。  
今年の後半戦から久々に業務にてiOSアプリを開発することが決まっていたため、実に1年半ぶりにSwiftを扱うことになりました。  

Swift4を積極的に利用していた際に、便利と名高い `Codable` に触れる機会があったため、  
本記事では実際に使ってみた `Codable` の例を書きたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### Codableとは

これまで `JSON` をパースするためには、  
`JSONSerialization` や `SwiftyJSON` などのOSSライブラリを利用して値を取り出す形で書いていたことと思います。  

```objective-c
// これまでの例

import SwiftyJSON

// レスポンスデータ用の構造体
struct Restaurant {
    var id: String
    var name: String
    var latitude: Double
    var longitude: Double
}

// データ取得クラス
class HogeHoge {

    func fetchData() {

      // 省略 (API等でデータを取得)

      do {
          let json = try JSON(data: response.data)

          // APIから返却されたjsonデータが以下だったとします。
          // {
          //    "id": "1"
          //    "name": "hogehoge"
          //    "latitude": "37"
          //    "longitude": "135"
          // }

          let id = json["id"].string ?? "ID不明"
          let name = json["name"].string ?? "ショップ名不明"
          let latitude = json["latitude"].string ?? "0"
          let longitude = json["longitude"].string ?? "0"
          let restaurant = Restaurant(id: id, name: name, latitude: latitude, longitude: longitude)
      } catch {
          ...
      }
    }
}
```

上記のように、 `SwiftyJSON` を利用して `JSON` 化したオブジェクトから、  
`let id = json["id"].string` や `let name = json["name"].string` のように返却レスポンスである `Restaurant` に自身でマッピングする必要がありました。
これがなんと、 `Codable` を利用することで、毎回のように書いていたマッピング処理が簡素化されるようです。  

### 実際の利用例
今回はFacebookログインによる認証処理が機能の1つとして必要でした。  
また、Facebookの `Graph API` を叩いて幾つかのプロフィール情報を取得するのですが、  
その中で `gender` の項目のみ「Facebookが返却する値」と「こちらのシステムが担保する `gender` の値」が異なっていました。  

形式の違いは以下の通りです。  
まずは、Facebookが返却するgenderの値です。  

```objective-c
男性の場合 → "male"
女性の場合 → "female"
```

続いて必要だった形式です。  

```objective-c
男性の場合 → 0
女性の場合 → 1
```

もちろん上記違いがあったとしても `Codable` は利用可能ですので、実際に書いてみました。  
※ Facebookログインを利用するための前準備は [SwiftでFacebookログインを始めよう！](http://grandbig.github.io/blog/2016/05/14/facebook-login-for-swift/) を参考にしてください。  

#### システム固有の性別のEnumの定義

```objective-c
// 性別のEnum定義
public enum Gender: Int {
    case male
    case female
}
```

#### Codableに準拠する構造体の定義

```objective-c
// Facebookから取得するユーザ情報の構造体 (Codableに準拠)
public struct FacebookUserProfile: Codable {

    public var id: String     // FacebookID
    public var email: String? // Facebookに登録されているメールアドレス
    public var gender: Gender // Facebookに登録されている性別

    // APIとアプリ間でありがちなプロパティ名の違いをココで吸収できる
    // 今回は全て同じなので問題なし
    private enum CodingKeys: String, CodingKey {
        case id = "id"
        case email
        case gender
    }

    // デコード処理をカスタマイズ
    public init(from decoder: Decoder) throws {
        let values = try decoder.container(keyedBy: CodingKeys.self)
        self.id = try values.decode(String.self, forKey: .id)
        self.email = try values.decodeIfPresent(String.self, forKey: .email)

        // APIから取得した性別の値をこちらのシステムの形式に変更する
        let genderString = try values.decode(String.self, forKey: .gender)
        if genderString == "male" {
            self.gender = .male
        } else {
            self.gender = .female
        }
    }

    // エンコード処理をカスタマイズ
    public func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(id, forKey: .id)
        try container.encodeIfPresent(email, forKey: .email)

        // システム固有の形式をAPIから取得した形式に戻す
        if gender == .male {
            try container.encode("male", forKey: .gender)
        } else {
            try container.encode("female", forKey: .gender)
        }
    }
}
```

#### JSONDecoderの拡張定義
少々困ったのが、Facebookの `Graph API` から返却される `response` を `data` 型に変換するメソッドが予め用意されていないことでした。  
毎回、 `JSONSerialization` を利用する処理を1行加えるのも手間なので、以下のように拡張メソッドを用意することにしました。  

```objective-c
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

#### Facebook Graph APIによるプロフィール情報取得処理
準備は整ったので、実際に `Graph API` を叩く部分の処理を実装します。  

```objective-c
import Foundation
import PromiseKit
import FacebookCore
import FacebookLogin

public final class FacebookManager {

    public static let shared = FacebookManager()
    private init() {
    }

    enum FacebookError: Error {
        case cancel       // Facebook認証をキャンセルした場合
        case apiError     // APIからエラーが返却された場合
        case decodeError  // デコード処理に失敗した場合
    }

    public enum FacebookUserField: String {
        case id = "id"
        case email = "email"
        case gender = "gender"
    }

    /// Facebookからユーザのプロフィール情報を取得する
    ///
    /// - Parameter facebookUserFields: 取得するプロフィール情報の指定
    /// - Returns: ユーザのプロフィール情報
    public func getUserProfile(facebookUserFields: [FacebookUserField] = [.id, .email, .gender]) -> Promise<FacebookUserProfile> {

        let promise = Promise<FacebookUserProfile> { fulfill, reject in
            let fields = facebookUserFields.map { $0.rawValue }
            let field = fields.joined(separator: ",")

            GraphRequest(graphPath: "me",
                         parameters: ["fields": field],
                         accessToken: AccessToken.current,
                         httpMethod: .GET,
                         apiVersion: .defaultVersion).start { response, result in

                            switch result {
                            case .success(let response):
                                do {
                                    let decoder = JSONDecoder()
                                    let profile = try decoder.decode(FacebookUserProfile.self,
                                                                     withJSONObject: response.dictionaryValue as Any)
                                    _ = fulfill(profile)
                                } catch {
                                    _ = reject(FacebookError.decodeError)
                                }
                            case .failed(let error):
                                print(error.localizedDescription)
                                _ = reject(FacebookError.apiError)
                            }
            }
        }

        return promise
    }
}
```

上記の通り、 `let profile = try decoder.decode(...)` だけで済んでいますね。  
今回は1つのメソッドしか書いていませんが同様のレスポンスを持つ複数メソッドを定義する必要がある場合などにより効力を発揮しそうな気がします。  

### まとめ
さて如何でしたでしょうか？  
`Codable` は今後更なる進化も予定されていると聞きますし、楽しみな機能の1つですね。  
筆者も引き続き積極的にSwift4の機能を利用していきたいと思います。  

最後にソースコード全体を載せておきます。  
(説明上、関係のないところは割愛していたので...)  

```objective-c
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
    }

    public enum FacebookUserField: String {
        case id = "id"
        case email = "email"
        case gender = "gender"
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
                      permissions: [FacebookPermission] = [.email, .publicProfile]) -> Promise<String> {

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

    /// Facebookからユーザのプロフィール情報を取得する
    ///
    /// - Parameter facebookUserFields: 取得するプロフィール情報の指定
    /// - Returns: ユーザのプロフィール情報
    public func getUserProfile(facebookUserFields: [FacebookUserField] = [.id, .email, .gender]) -> Promise<FacebookUserProfile> {

        let promise = Promise<FacebookUserProfile> { fulfill, reject in
            let fields = facebookUserFields.map { $0.rawValue }
            let field = fields.joined(separator: ",")

            GraphRequest(graphPath: "me",
                         parameters: ["fields": field],
                         accessToken: AccessToken.current,
                         httpMethod: .GET,
                         apiVersion: .defaultVersion).start { response, result in

                            switch result {
                            case .success(let response):
                                do {
                                    let decoder = JSONDecoder()
                                    let profile = try decoder.decode(FacebookUserProfile.self,
                                                                     withJSONObject: response.dictionaryValue as Any)
                                    _ = fulfill(profile)
                                } catch {
                                    _ = reject(FacebookError.decodeError)
                                }
                            case .failed(let error):
                                print(error.localizedDescription)
                                _ = reject(FacebookError.apiError)
                            }
            }
        }

        return promise
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

といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
