---
layout: post
title: "ぐるなびレストラン検索APIで困ったCodable"
date: 2018-05-03 00:37
comments: true
categories: ios swift swift4 codable
---

### はじめに
さて、今回は `Codable` で困った話です。  
[ぐるなびレストラン検索API](http://api.gnavi.co.jp/api/manual/restsearch/)を利用する機会がありまして、  
どうせなら `Codable` を使ってキレイに書こうと思ったが...という話になります。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### ぐるなびレストラン検索APIのレスポンスについて
ほとんどの場合は何も考えずに `Codable` が適用できたのですが、`image_url`だけ困りました。  
と言うのも返却されるレスポンスに以下の違いがあったためです。  

実際にAPIを叩いて頂くと...  

```objective-c
// 値がある場合
image_url {
    shop_image1: http\:\/\/xxxxxxx,
    shop_image2: http\:\/\/xxxxxxx,
    qrcode: http\:\/\/xxxxxxx
}

// 値がない場合
image_url {
    shop_image1: {
    },
    shop_image2: {
    },
    qrcode: http\:\/\/xxxxxxx
}
```

といった感じになっていることがわかると思います。  

### Codableをどこまで利用するべきか
筆者が下した結論としては、全てを `Codable` だけで賄うのは難しいということです。  
なぜなら `Codable` は `Any` 型を扱えないからです。  
ではどうすれば良いかと言うと...  

* `SwiftyJSON`を使う  
* `Codable` と `SwiftyJSON` を使う  

ことが考えられるかと思います。  

せっかくなので、`Codable` と `SwiftyJSON` を両方使ってみた場合を見ていきましょう。  

まず`Codable`の定義です。  

```objective-c
public struct Restaurant: Codable {

    public var id: String
    public var name: String
    public var nameKana: String
    public var address: String
    public var url: String
    public var tel: String
    public var latitude: String
    public var longitude: String
    public var urlMobile: String
    public var shopImage1: String?
    public var shopImage2: String?
}

public struct Restaurants: Codable {

    public var rest: [Restaurant]
    public var totalHitCount: String
    public var hitPerPage: String
    public var pageOffset: String
}
```

続いて、`SwiftyJSON`をどのように使ったかというと...  
(以下、処理の抜粋です。)  

```objective-c
Alamofire.request(requestURL, method: .get, parameters: parameters, encoding: URLEncoding.default, headers: nil)
    .response { response in
        do {
            var result = Restaurants(rest: [], totalHitCount: "", hitPerPage: "", pageOffset: "")

            guard let data = response.data else {
                return
            }
            let rest = JSON(data)["rest"]
            let decoder = JSONDecoder()
            decoder.keyDecodingStrategy = .convertFromSnakeCase
            let restaurants = try decoder.decode(Restaurants.self, from: data)
            guard let array = rest.array else {
                completion(restaurants)
                return
            }
            for elem in array {
                for restaurant in restaurants.rest where restaurant.id == elem["id"].string {
                    var convertedRestaurant = restaurant
                    convertedRestaurant.shopImage1 = elem["image_url"]["shop_image1"].string
                    convertedRestaurant.shopImage2 = elem["image_url"]["shop_image2"].string
                    result.rest.append(convertedRestaurant)
                    break
                }
            }

            completion(result)
        } catch {
            print("error")
        }
}
```

ただ、これは`for`文をぶん回して照らし合わせて`Codable`の形に充てがいたいがために書いているので、処理効率が良いかは別の話ですね...  
APIで取得できた件数が少なければ大した話ではありませんが...。  
(大量件数が取得されるケースでは、むしろ少数に減らすことができないか検討すべきでしょう。)  

### まとめ
さて、今回は`Codable`を利用しようと思って困った話を書いてみました。  
今回のAPIの返却値は特殊な感じもするので、今後Swiftが対応していくようになるとは思えないのですが、  
もしかしたらもう少し書きやすくなることもあるかもしれません。  

逆に、iOSアプリ向けにAPIを新規開発するときは、上記のような事情も踏まえて開発すると困ることが少ないのではと思います。  

と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
