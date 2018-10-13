---
layout: post
title: "iOSで気をつけるGoogle Places APIの使い分け"
date: 2018-05-27 18:58
comments: true
categories: ios swift google
---

### はじめに
今回は[Places API for iOS](https://developers.google.com/places/ios-api/?hl=ja)と[Google Places API](https://developers.google.com/places/?hl=ja)を使い分けた話です。

`Google Places API` はデバイスやOSに依らず用意されたAPIですが、 `Places API for iOS` はネーミングからも分かる通り `iOS` のために用意された `API` という位置づけになります。  
通常であれば、「 `Places API for iOS` を使えば良いのでは？」と思われるかもしれませんが、  
筆者がプロダクト開発時に実現したかった内容がたまたまそぐわなかったので `Google Places API` を利用するに至ったわけです。  

では早速見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### 周辺の場所情報を取得する
現在の場所を取得するのであれば、Googleが[スタートガイド](https://developers.google.com/places/ios-api/start?hl=ja)で説明しているように、  
`GMSPlacesClient` の `currentPlaceWithCallback` メソッドを利用すれば可能です。  

しかし、特定の場所を起点に周辺の場所情報を取得する場合は、`Google Places API` を利用する方が汎用性があります。  
`Places API for iOS` にも `Place Picker` が用意されていますが、以下理由より用途が限定的に感じられました。  

* UI/UXが限定されている  
* 特定の属性のものだけ検索することに向いていない  

そこで筆者は、「周辺の病院を検索する」ケースでは `Google Places API` を利用することにしました。  
以下に具体的な実装を記載します。  
通信ライブラリとして `Moya` を非同期処理のために `PromiseKit` を利用します。  

```objective-c
// HospitalAPI.swift
import Foundation
import Moya
import PromiseKit
import GooglePlaces

internal enum HospitalAPITarget {
    case hospitals(lat: Double, lng: Double)
}

internal enum APIError: Error {
    case cancel
    case apiError(description: String)
    case decodeError
}

internal enum GooglePlacesError: Error {
    case cancel
    case notFoundError
}

extension HospitalAPITarget: TargetType {

    /// API Key
    private var apiKey: String {
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
        switch self {
        case .hospitals:
            return R.string.url.googlePlacesApiPlaceUrl()
        }

    }

    public var baseURL: URL {
        return URL(string: _baseURL)!
    }

    // enumの値に対応したパスを指定
    public var path: String {
        switch self {
        case .hospitals:
            return ""
        }
    }

    // enumの値に対応したHTTPメソッドを指定
    public var method: Moya.Method {
        switch self {
        case .hospitals:
            return .get
        }
    }

    // スタブデータの設定
    public var sampleData: Data {
        switch self {
        case .hospitals:
            return "Stub data".data(using: String.Encoding.utf8)!
        }
    }

    // パラメータの設定
    var task: Task {
        switch self {
        case let .hospitals(lat, lng):
            return .requestParameters(parameters: [
                R.string.common.keyFileName(): apiKey,
                R.string.common.locationKeyName(): "\(lat),\(lng)",
                R.string.common.radiusKeyName(): 1500,
                R.string.common.typeKeyName(): "hospital"
                ], encoding: URLEncoding.default)
        }
    }

    // ヘッダーの設定
    var headers: [String: String]? {
        switch self {
        case .hospitals:
            return nil
        }
    }
}

class HospitalAPI: HospitalProtocol {
    private var provider: MoyaProvider<HospitalAPITarget>!

    /// イニシャライザ
    init() {
        provider = MoyaProvider<HospitalAPITarget>()
    }

    // MARK: CRUD operations

    /// 指定の緯度、経度から一定範囲内の病院を検索する処理
    ///
    /// - Returns: 病院のプレイス情報
    func fetchHospitals(lat: Double, lng: Double) -> Promise<[Place]> {
        let (promise, resolver) = Promise<[Place]>.pending()

        provider.request(.hospitals(lat: lat, lng: lng)) { result in
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

因みに、上記 `fetchHospitals` メソッド内で利用している `Place` の定義は以下です。  

```objective-c
// Place.swift
import Foundation

public struct Location: Codable {

    public var lat: Double
    public var lng: Double
}

public struct Viewport: Codable {

    public var northeast: Location
    public var southwest: Location
}

public struct Geometry: Codable {

    public var viewport: Viewport
    public var location: Location
}

public struct OpeningHours: Codable {

    public var weekdayText: [String]
    public var openNow: Bool
}

public struct Photos: Codable {

    public var photoReference: String
    public var width: Double
    public var height: Double
    public var htmlAttributions: [String]
}

public struct Place: Codable {

    public var id: String
    public var placeId: String
    public var name: String
    public var icon: String
    public var rating: Double?
    public var scope: String
    public var vicinity: String
    public var reference: String
    public var priceLevel: Int?
    public var types: [String]
    public var geometry: Geometry
    public var openingHours: OpeningHours?
    public var photos: [Photos]?
}

public struct Places: Codable {

    public var results: [Place]
    public var status: String
    public var htmlAttributions: [String]
}
```

### 場所の画像を取得する
Webクライアントから場所の画像を取得する場合は先程APIを叩いて取得した結果の `photos` からAPIを叩く流れになります。  
しかしながら、iOSでは画像取得用のメソッドが用意されています。  
このメソッドは `プレイスID` さえ指定すれば簡単に取得できるため、  
ここは適材適所で `Places API for iOS` の `lookUpPhoto` および `loadPlacePhoto` を利用した方が良いでしょう。  

```objective-c
// HospitalAPI.swift
func fetchPhoto(placeId: String) -> Promise<UIImage?> {
    let (promise, resolver) = Promise<UIImage?>.pending()

    GMSPlacesClient.shared().lookUpPhotos(forPlaceID: placeId) { (photos, error) in
        if let error = error {
            resolver.reject(error)
            return
        }
        guard let firstPhoto = photos?.results.first else {
            resolver.reject(GooglePlacesError.notFoundError)
            return
        }
        GMSPlacesClient.shared().loadPlacePhoto(firstPhoto, callback: { (image, error) in
            if let error = error {
                resolver.reject(error)
                return
            }
            resolver.fulfill(image)
        })
    }

    return promise
}
```


### まとめ
今回筆者が改めて学んだのは、iOSアプリを開発するからといって、`Places API for iOS` だけに焦点を絞るのではなく、  
『結局、何がしたくて、そのためにどういった情報を取得したいのか』
をきちんど整理した上で適切な手段を用いるということです。  

Googleが提供しているAPIは勉強になりますね。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
