---
layout: post
title: "PromiseKitを使ってみよう！(2)"
date: 2018-04-24 23:51
comments: true
categories: ios swift promise
---

### はじめに
今回は久しぶりに[PromiseKit](https://github.com/mxcl/PromiseKit)について書きたいと思います。  
今から約2年前に[PromiseKitを使ってみよう！](http://grandbig.github.io/blog/2016/04/09/c/)で少し触れていたのですが、  
最近業務で扱うことも増えてきたので改めて書き留めておこうと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### Promiseとは
`PromiseKit` に記載されている内容を訳すと(若干、意訳してますが...)、  

* シンプルな非同期プログラミングを実現できる  
* 上記によって、開発者がより重要な課題に集中することができる  
* 学習ハードルが低いため、マスターすることが簡単である  
* 可読性の高いコードが実現できるため、チーム開発にも向いている  

といった感じです。  
これを見る限り、非常に期待できますよね。  
では、具体的な使い方を見ていきましょう。  

### アラートでの利用例
`UIAlertViewController` を利用した時に、通常は以下のように `completion` を引数に持って実装するかと思います。  

```objective-c
internal func showConfirm(title: String,
                          message: String,
                          okCompletion: @escaping (() -> Void),
                          cancelCompletion: @escaping (() -> Void)) {
    let alert = UIAlertController.init(title: title, message: message, preferredStyle: UIAlertControllerStyle.alert)
    let okAction = UIAlertAction.init(title: "OK", style: UIAlertActionStyle.default) { _ in
        okCompletion()
    }
    let cancelAction = UIAlertAction.init(title: "キャンセル", style: UIAlertActionStyle.cancel) { _ in
        cancelCompletion()
    }
    alert.addAction(okAction)
    alert.addAction(cancelAction)
    present(alert, animated: true, completion: nil)
}
```

これだと引数が多くなってしまいますし、呼び出し側でも下記のように書くことになります。  

```objective-c
self.showConfirm(title: "確認", message: "OKとキャンセルどちらをタップしますか", okCompletion: {
    // OKボタンをクリックした場合に呼び出される
}) {
    // キャンセルボタンをクリックした場合に呼び出される
}
```

これを `PromiseKit` を使って書くとどうなるでしょうか？  
まずは、アラートのメソッドは下記のように書けます。  

```objective-c
import PromiseKit

internal enum AlertError: Error {
    case cancel
}

internal func showConfirm(title: String, message: String) -> Promise<Void> {
    let (promise, resolver) = Promise<Void>.pending()

    let alert = UIAlertController.init(title: title, message: message, preferredStyle: UIAlertControllerStyle.alert)
    let okAction = UIAlertAction.init(title: R.string.common.ok(), style: UIAlertActionStyle.default) { _ in
        resolver.fulfill(Void())
    }
    let cancelAction = UIAlertAction.init(title: R.string.common.cancel(), style: UIAlertActionStyle.cancel) { _ in
        resolver.reject(AlertError.cancel)
    }
    alert.addAction(okAction)
    alert.addAction(cancelAction)
    present(alert, animated: true, completion: nil)

    return promise
}
```

呼び出し側では、  

```objective-c
import PromiseKit

firstly {
  showConfirm(title: "確認", message: "OKとキャンセルどちらをタップしますか")
}.done { _ in
    // OKをタップした場合に呼び出される処理
}.catch { [weak self] error in
    // キャンセルをタップした場合に呼び出される処理
}
```

と書くことができ、非常にわかりやすく後続の処理を書くことができます。  

### API呼び出しの利用例
続いて、API呼び出しの場合の利用例を見ていきましょう。  
今回API通信処理では [Moya](https://github.com/Moya/Moya) を利用します。  
また、叩くAPIは [Google Places API](https://developers.google.com/places/?hl=ja) を利用します。    

まずは `Moya` の書き方ですが、  

```objective-c
import Foundation
import Moya
import PromiseKit

// APIエラーの定義
internal enum APIError: Error {
    case cancel
    case apiError(description: String)
    case decodeError
}

internal enum SampleAPITarget {
    case places(lat: Double, lng: Double)
}

extension SampleAPITarget: TargetType {

    /// API Key
    private var apiKey: String {
        // 以下はGoogleのAPIキーをkey.plistで保持していると仮定した処理です。
        guard let path = Bundle.main.path(forResource: "key", ofType: "plist") else {
            fatalError("key.plistが見つかりません")
        }

        guard let dic = NSDictionary(contentsOfFile: path) as? [String: Any] else {
            fatalError("key.plistの中身が想定通りではありません")
        }

        guard let apiKey = dic["googleApiKey"] as? String else {
            fatalError("Google APIのKeyが設定されていません")
        }

        return apiKey
    }

    // ベースURLを文字列で定義
    private var _baseURL: String {
        return "https://maps.googleapis.com/maps/api/place/nearbysearch/json"
    }

    public var baseURL: URL {
        return URL(string: _baseURL)!
    }

    // enumの値に対応したパスを指定
    public var path: String {
        switch self {
        case .places:
            return ""
        }
    }

    // enumの値に対応したHTTPメソッドを指定
    public var method: Moya.Method {
        switch self {
        case .places:
            return .get
        }
    }

    // スタブデータの設定
    public var sampleData: Data {
        switch self {
        case .places:
            return "Stub data".data(using: String.Encoding.utf8)!
        }
    }

    // パラメータの設定
    var task: Task {
        switch self {
        case .places(let lat, let lng):
            return .requestParameters(parameters: [
                R.string.common.keyFileName(): apiKey,
                R.string.common.locationKeyName(): "\(lat),\(lng)",
                R.string.common.radiusKeyName(): 500
                ], encoding: URLEncoding.default)
        }
    }

    // ヘッダーの設定
    var headers: [String: String]? {
        switch self {
        case .places:
            return nil
        }
    }
}

class SampleAPI {

    private var provider: MoyaProvider<SampleAPITarget>!

    /// イニシャライザ
    init() {
        provider = MoyaProvider<SampleAPITarget>()
    }
}
```

のように書きます。  
( `Moya` はUnitテストが書きやすいので良いんですよね〜という話はまた後日... )  

では、本題の `PromiseKit` を用いたAPI通信処理です。  
先程の `SampleAPI` クラスにメソッドを追加してみます。  

```objective-c
class SampleAPI {

    private var provider: MoyaProvider<SampleAPITarget>!

    /// イニシャライザ
    init() {
        provider = MoyaProvider<SampleAPITarget>()
    }

    // Placeの定義は
    // https://grandbig.github.io/blog/2018/04/23/codable-swift4-1/
    // の『オマケ：Google Places APIでCodableを利用する』を参照のこと
    func fetchPlaces(lat: Double, lng: Double) -> Promise<[Place]> {
        let (promise, resolver) = Promise<[Place]>.pending()

        provider.request(.places(lat: lat, lng: lng)) { result in
            switch result {
            case .success(let response):
                do {
                    let decoder = JSONDecoder()
                    decoder.keyDecodingStrategy = .convertFromSnakeCase
                    let places = try decoder.decode(Places.self, from: response.data)

                    resolver.fulfill(places.results)
                } catch {
                    resolver.reject(APIError.decodeError)
                }
            case .failure(let error):
                resolver.reject(APIError.apiError(description: error.localizedDescription))
            }
        }

        return promise
    }
}
```

上記のように定義できます。  
これを呼び出すときは、  

```objective-c
var worker = SampleAPI()

firstly {
    worker.fetchPlaces(lat: latitude, lng: longitude)
}.done { [weak self] results in
    guard let strongSelf = self else { return }
    print(results)
}.catch { [weak self] error in
    guard let strongSelf = self else { return }
    print(error.localizedDescription)
}
```

のように書くことができます。  
こちらも縦に浅いネストで読むことができるため、可読性が高いと言えますね。  

### PromiseKitの公式Readme
`PromiseKit` のドキュメントは非常に丁寧なので、これを読むだけでも理解を相当深められると思います。   

* [GettingStarted](https://github.com/mxcl/PromiseKit/blob/master/Documentation/GettingStarted.md)  
* [CommonPatterns](https://github.com/mxcl/PromiseKit/blob/master/Documentation/CommonPatterns.md)  

を読んでおけばOKかと。  
(上記の話も別機会でかければと思います。)  

### まとめ
さて、如何でしたでしょうか？  
筆者は割りと `completion` で書くのが好きだったのですが、 `PromiseKit` に慣れていくと、その良さにどんどん気づいていくことができました。  
リトライ処理は遅延実行も簡単に対応できたりするので、ぜひ使ってみることをオススメします。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
