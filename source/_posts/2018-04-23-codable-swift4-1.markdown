---
layout: post
title: "Swift4.1で利便性が向上したCodable"
date: 2018-04-23 00:00
comments: true
categories: ios swift swift4 codable
---

### はじめに
今回はSwift4.1でさらに利便性が向上した `Codable` について紹介したいと思います。  
Swift4で登場した `Codable` ですが、何がより便利になったかというと、 **スネークケース** から **キャメルケース** への変換処理です。  

早速、実例を交えて何が変わったのか見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### スネークケースからキャメルケースへの変換処理
まずは、説明のために扱うAPIについて説明したいと思います。  

#### サンプルとして利用するAPIについて
今回は、 [Google Places API](https://developers.google.com/places/web-service/?hl=ja)を元に説明します。  
`Google Places API` で返却されるレスポンスの一例は以下の通りです。  

![Google Places APIのレスポンスの一部](/images/swift4_1_codable.png)  

#### Swift4での対応方法
これまでのSwift4では以下のようにスネークケースからキャメルケースへの対応をしていました。  

```objective-c
public struct Place: Codable {

    public var id: String
    public var placeId: String
    public var name: String
    ...

    private enum CodingKeys: String, CodingKey {
        case id
        case placeId = "place_id"
        case name
    }
}
```

上記のように、自身で `CodingKey` プロトコルに準拠した `CodingKeys` を定義していました。  

#### Swift4.1での対応方法
スネークケースからキャメルケースへの変換はほぼほぼ確実に発生しうる命名規則と考えられるので対応したのでしょうか？  
Swift4.1では実に簡単に対応できます。  

```objective-c
public struct Place: Codable {

    public var id: String
    public var placeId: String
    public var name: String
    ...
}
```

Swift4.1では `CodingKeys` 内で設定する必要がありません。  
サーバレスポンスの変換前に以下の1行を追加するだけで自動で判断してくれるようになりました。  

```objective-c
// Moyaを利用したAPIレスポンスを扱う例
// 今回はMoyaが説明のメインではないため、深くは説明しない
provider.request(.places(lat: lat, lng: lng)) { result in
    switch result {
    case .success(let response):
        do {
            let decoder = JSONDecoder()

            // 下記を追加
            decoder.keyDecodingStrategy = .convertFromSnakeCase
            // 上記を追加

            let places = try decoder.decode(Places.self, from: response.data)
            print("places: \(places)")
        } catch {
            print("error: 変換エラー")
        }
    case .failure(let error):
        print("error: \(error)")
    }
}
```

### オマケ：Google Places APIでCodableを利用する
今回説明するにあたって定義した `Google Places API` 用の `Codable` を折角なので書いておきたいと思います。  

```objective-c
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

    public var places: [Place]
    public var status: String
    public var htmlAttributions: [String]

    private enum CodingKeys: String, CodingKey {
        case places = "results"
        case status
        case htmlAttributions
    }
}
```

### まとめ
さて如何でしたでしょうか？  
オープンソース化された効果なのか、Swiftも現場で必要とされる使われ方にどんどん適応していっている感じがしますね。  
今後に更なる期待を寄せつつ、本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
