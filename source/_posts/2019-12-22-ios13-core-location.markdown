---
layout: post
title: "iOS13におけるCoreLocationの変更点〜常に許可/使用中のみ許可/一時的に許可〜"
date: 2019-12-22 00:00
comments: true
categories: ios ios13 location
---

### はじめに
「[iOS #2 Advent Calendar 2019](https://qiita.com/advent-calendar/2019/ios-2)」の22日目の記事です。  

筆者がiOSアプリの開発を始めたのは、約8年ほど前でしょうか。  
iOS史上でも初期から強力な機能だったこともあってか、当時はiOSアプリに位置情報の機能を載せることが流行っており、  
筆者も漏れなく位置情報に関する調査や機能実装を永遠とこなしていた気がします。  

今年でiOSもバージョン `13` となり、昔はなかった機能がたくさん登場しています。  
新しい機能は当然、開発者の意欲を掻き立て、未知の世界をユーザに届けることに寄与することでしょう。  
ただ、位置情報に人一倍強い思いがあることを自負している筆者ですから、  
iOS13が出た今でも位置情報に関する仕様変更があることには感動もひとしおです。  

今日は、iOS13からの位置情報に関する仕様変更を『[WWDC2019 - What's New in Core Location](https://developer.apple.com/videos/play/wwdc2019/705/)』を元に紹介して1年を締めくくりたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### 位置情報とプライバシーについて
約8年前は位置情報に関する様々な国内での取り決め事がまだまだ調整途中だったかと思います。  
ただし、当時から「電池の消耗が激しいこと」と同じくらい「自身の位置情報を知られたくない」というプライバシーに対する強い意識があった気がします。  
(もちろん現在の方がプライバシー意識は劇的に高まっていると思います。)  

一方で、企業側としては、位置情報を利用した新しい便利な体験をユーザに提供するために、  
位置情報を積極的に許可してほしい気持ちがあったかと思います。   

これまでiOSはそんなユーザとアプリ提供者側の双方の立場を考慮して改善を進めてきました。  
例えば、初め「常に許可」「許可しない」の2つの選択肢しかなかったところに、  
「使用中のみ許可」が加わったりといった取り組みです。  

iOS13では、この取り組みが更に一歩進んだ形になります。  

### 位置情報の利用確認ダイアログに「常に許可」の選択肢がない
iOS12までは、  

```objective-c
var locationManager = CLLocationManager()

locationManager.delegate = self

locationManager.requestAlwaysAuthorization()
```

とした際に、

* 「常に許可」  
* 「使用中のみ許可」  
* 「許可しない」  

の3つの選択肢を持つ確認ダイアログが表示されていました。  

これがiOS13では、

* 「使用中のみ許可」  
* 「一時的に許可」  
* 「許可しない」  

の3つの選択肢に変更となりました。  
一見すると、アプリ開発者にとって辛い仕様に思えるのですが、「常に許可」がなくなったわけではありません。  

**「常に許可」を求めるタイミングが変更になった**

というのが正しい解釈となります。  

因みに、ここで「使用中のみ許可」を選択した場合、  
`CLAuthorizationStatus` は実は `.authorizedAlways` になります。  

### 「常に許可」に変更するタイミング
どういうことなのか詳しく説明しましょう。  

先程の「使用中のみ許可 / 一時的に許可 / 許可しない」の3つの選択肢から **「使用中のみ許可」** を選択した場合、OSがユーザが忙しくない時を自動的に狙って、

* 「使用中のみ許可のままにする」  
* 「常に許可に変更する」  

かを問いかけてくれます。  

![常に許可に変更する確認ダイアログ](/images/ios13-core-location_1.jpg)  

ここで「常に許可に変更する」を選択して初めて、「常に許可」状態に設定することができます。  

この仕様がアプリ提供元の企業やアプリ開発者に対して、何を問うているのでしょうか。  

筆者は、  

* ユーザに「アプリの利用価値を伝え、位置情報を利用することの有用性を体感してもらう」ことが必要  
* ユーザに「アプリで利用する位置情報は、プライバシーが守られていることを伝える」ことが必要  

という大前提に加えて、  

* アプリが最大価値を発揮するのは、Foreground起動中であるべき  
* ユーザがアプリを初めて利用する場面は、必ずForeground起動であるはず  

という釘を差しているようにも捉えています。  

後者は、OSや純正アプリによって担保されていますが、前者はAppleの審査やそもそもの規約があるものの、作り手の仕組みにも依存する側面は完璧には拭えないでしょう。  

因みに、  
「使用中のみ許可のままにする / 常に許可に変更する」かの確認は1度のみであるため、  
これを逃すと、ユーザが自主的に設定画面から設定を変更しない限り「常に許可」に変更する導線がなくなります。  

### 「使用中のみ許可」の変更点
iOS12ではユーザが「使用中のみ許可」を選択した場合、下記のようにできることが限られていました。  

* 位置情報を取得すること  
* iBeaconを監視すること  
* Backgroundで位置情報を継続して取得すること  

もし、ユーザが「常に許可」を選択すれば、上記に加えて下記も利用することができるようになります。  

* 位置情報の取得の「開始」  
* 大幅変更位置情報サービスを利用すること  
* 領域監視を利用すること  
* 滞在情報監視を利用すること  

![iOS12: 常に許可/使用中のみ許可の違い表](/images/ios13-core-location_2.png)  
※WWDC2019の資料から該当箇所を抜粋して紹介させて頂いています。
https://developer.apple.com/videos/play/wwdc2019/705/

このため、アプリ開発者は、『どんな機能をアプリに持たせたいか』次第で「常に許可」「使用中のみ許可」のどちらをユーザに求めるべきかを決めていました。  

これがiOS13では非常にシンプルになりました。  

![iOS13: 常に許可/使用中のみ許可の違い表](/images/ios13-core-location_3.png)  
※WWDC2019の資料から該当箇所を抜粋して紹介させて頂いています。
https://developer.apple.com/videos/play/wwdc2019/705/

つまり、機能間の差異をなくして、『ユーザに求めた許可状態に即したタイミングで機能を提供する』ことが可能になったということです。  

「常に許可」な状態はアプリがForeground/Background起動に関わらず、常に機能を利用できる状態であることがわかるかと思いますが、  
「使用中のみ許可」な状態とは具体的にどんな状態を指すのでしょうか。  

ここで簡単なサンプルアプリを作成して説明してみます。  

### 「使用中のみ許可」の「使用中」の状態とは
下記のような簡単なサンプルアプリを作成してみました。  

#### 機能
実験のためのアプリなので、機能は下記のみです。  

* 「常に許可」設定をユーザに求める  
* 「使用中のみ許可」設定をユーザに求める  
* BackgroundモードはOFFで位置情報取得を開始する
* BackgroundモードはOFFで位置情報取得を終了する  
* BackgroundモードはONで位置情報取得を開始する
* BackgroundモードはONで位置情報取得を終了する  

#### 画面キャプチャ
実際の画面は下記の通りです。  

![サンプルアプリの画面](/images/ios13-core-location_1.jpg)  

#### ソースコード
実際のソースコードは下記の通りです。  

```objective-c
import UIKit
import CoreLocation

class ViewController: UIViewController {
  // MARK: - Properties
  var locationManager = CLLocationManager()

  // MARK: - Lifecycle
  override func viewDidLoad() {
    super.viewDidLoad()

    configureLocationManager()
  }

  // MARK: - Configure
  private func configureLocationManager() {
    locationManager.delegate = self
    locationManager.desiredAccuracy = kCLLocationAccuracyBestForNavigation
    locationManager.activityType = CLActivityType.fitness
    locationManager.pausesLocationUpdatesAutomatically = false
  }

  // MARK: - IBActions
  @IBAction private func didTapRequestAlwaysAuthorizationButton(_ sender: Any) {
    locationManager.requestAlwaysAuthorization()
  }

  @IBAction private func didTapRequestWhenInUseAuthorizationButton(_ sender: Any) {
    locationManager.requestWhenInUseAuthorization()
  }

  @IBAction private func didTapStartUpdatingLocationButton(_ sender: Any) {
    locationManager.startUpdatingLocation()
  }

  @IBAction private func didTapStopUpdatingLocationButton(_ sender: Any) {
    locationManager.stopUpdatingLocation()
  }

  @IBAction private func didTapStartUpdatingLocationAllowsBackgroundButton(_ sender: Any) {
    locationManager.allowsBackgroundLocationUpdates = true
    locationManager.startUpdatingLocation()
  }

  @IBAction private func didTapStopUpdatingLocationAllowsBackgroundButton(_ sender: Any) {
    locationManager.allowsBackgroundLocationUpdates = false
    locationManager.stopUpdatingLocation()
  }
}

// MARK: - CLLocationManagerDelegate
extension ViewController: CLLocationManagerDelegate {

  func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
    switch status {
    case .notDetermined:
      print("未決定の場合")
    case .authorizedAlways:
      print("常に許可した場合")
    case .authorizedWhenInUse:
      print("使用中のみ許可した場合")
    case .denied:
      print("許可しない場合")
    case .restricted:
      print("位置情報を利用できない制限がある場合")
    @unknown default:
      fatalError("CLAuthorizationStatusの種類が増えているので、条件を見直す必要があります。")
    }
  }

  func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
    guard let location = locations.first else { return }
    print("location: \(location.coordinate)")
  }
}
```

#### 実験
では、具体的に「使用中」状態を見るための実験をしてきましょう。  

##### 実験１：BackgroundモードOFFの位置情報取得
まずは、「使用中のみ許可」設定をしてから、 **BackgroundモードOFFの位置情報取得開始** を実行してみましょう。  

```objective-c
// 使用中のみ許可を求める
locationManager.requestWhenInUseAuthorization()

// 位置情報の取得を開始する
locationManager.startUpdatingLocation()
```

この結果、アプリがForeground起動の場合、 `didUpdateLocations` が呼び出され、最新の位置情報を取得できます。  
しかし、アプリをBackground起動にした場合、 `didUpdateLocations` が呼び出されることはなくなりました。  

※タイミングによっては呼び出されることがありますが、それはアプリはBackgroundで数秒Foregroundと同じ扱いになるOS仕様のためです。  

上記の例では、この **Foreground起動中が「使用中」** に当たります。  

そのため、再び、Foreground起動にアプリを戻すと「使用中」状態に戻るため、 `didUpdateLocations` が呼び出されるようになります。  

##### 実験２：BackgroundモードONの位置情報取得
続いて、 **BackgroundモードONの位置情報取得開始** を実行してみましょう。  

```objective-c
// 使用中のみ許可を求める
locationManager.requestWhenInUseAuthorization()

// Backgroundでの位置情報の更新を許可する
locationManager.allowsBackgroundLocationUpdates = true
// 位置情報の取得を開始する
locationManager.startUpdatingLocation()
```

このとき、アプリがForeground起動の場合、 `didUpdateLocations` が呼び出され、最新の位置情報を取得できます。  
それに加えて、アプリをBackground起動にした場合でも、 `didUpdateLocations` が呼び出され続けます。  

この場合の例では、 **ForegroundおよびBackgroundともに「使用中」** に当たります。  

Backgroundで位置情報を利用していることが、ユーザにも伝わるように、ステータスバーの左側に青色で囲まれた矢印マークが表示されます。  

![Backgroundで位置情報を利用している状態](/images/ios13-core-location_5.jpg)  

### 「使用中のみ許可」ユーザが使用中でない場合へのアプローチ
これで **使用中** とはどういった状態を指すのか、おわかり頂けたかと思います。  

基本的にはアプリはユーザが利用したい時に利用することになると思いますが、  
アプリ提供元がアプリを利用する最適な場面を知らせたいといったこともあるでしょう。  

そういった時には、 `UNLocationNotificationTrigger` を使ったジオフェンスによるローカルプッシュが役に立つかもしれません。  

特定の場所にジオフェンスを仕掛けておくことで、  
領域侵入 or 領域退出時にローカルプッシュでユーザを気づかせ、  
アプリを起動してもらうことで **使用中** 状態に誘導し、更なる価値提供を狙えることでしょう。  

参考までに `UNLocationNotificationTrigger` の利用方法も記載しておきます。  

まずは、 `AppDelegate.swift` で最低限の準備をしておきましょう。  

```objective-c
// AppDelegate.swift
class AppDelegate: UIResponder, UIApplicationDelegate {

  func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    let center = UNUserNotificationCenter.current()
    center.delegate = self

    // 通知の許可を求める
    center.requestAuthorization(options: [.sound, .alert, .badge]) { (result, error) in
      print(result)
    }
    return true
  }

  ...

}

// MARK: - UNUserNotificationCenterDelegate
extension AppDelegate: UNUserNotificationCenterDelegate {

  // 未起動 or Background起動時にローカルプッシュを受信した場合
  func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
    completionHandler()
  }

  // Foreground起動時にローカルプッシュを受信した場合
  func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
    completionHandler(.alert)
  }
}
```

続いて、ジオフェンスを設定します。  

```objective-c
// ViewController.swift
class ViewController: UIViewController {
    ...
    private func configureLocationNotificationTrigger() {
        let content = UNMutableNotificationContent()
        content.title = "お知らせ！"
        content.body = "今、アプリを利用するとお得な情報をGETできます！"
        content.sound = UNNotificationSound.default

        let center = CLLocationCoordinate2D(latitude: 35.0, longitude: 139.7)
        let region = CLCircularRegion(center: center, radius: 200, identifier: "特定の場所")
        region.notifyOnEntry = true
        region.notifyOnExit = true
        let trigger = UNLocationNotificationTrigger(region: region, repeats: true)
        let request = UNNotificationRequest(identifier: "ジオフェンス", content: content, trigger: trigger)
        let notificationCenter = UNUserNotificationCenter.current()
        notificationCenter.add(request)
    }
}

// MARK: - CLLocationManagerDelegate
extension ViewController: CLLocationManagerDelegate {

  func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
    switch status {
    case .notDetermined:
      print("未決定の場合")
    case .authorizedAlways:
      print("常に許可した場合")
    case .authorizedWhenInUse:
      print("使用中のみ許可した場合")
      configureLocationNotificationTrigger()
    case .denied:
      print("許可しない場合")
    case .restricted:
      print("位置情報を利用できない制限がある場合")
    @unknown default:
      fatalError("CLAuthorizationStatusの種類が増えているので、条件を見直す必要があります。")
    }
  }
  ...
}
```

上記のように、「使用中のみ許可」の場合にのみ `UNLocationNotificationTrigger` を仕掛けておくことも一つの手です。  

「常に許可」は既にユーザから最大限の承諾を得ているため、更なる価値を届けるためのプッシュ通知という意味では特に意味を持たないためです。  

### 「一時的に許可」とは
さて、ここまで「常に許可」「使用中のみ許可」について説明してきましたが、  
iOS13からは「一時的に許可」という選択肢が新たに追加になりました。  

「一時的に許可」とは文字通り、 **その時の使用中の間のみ許可をする** ということです。  
よって、 `CLAuthorizationStatus` も `.authorizedWhenInUse` になります。  

また、 **その時の使用中の間のみ許可する** の「使用中のみ」は、「使用中のみ許可」と同じ定義になります。  

つまり、  

* `allowsBackgroundLocationUpdates=false` で `locationManager.startUpdatingLocation()` するとForeground起動中が「使用中」に当たります  
* `allowsBackgroundLocationUpdates=true` で `locationManager.startUpdatingLocation()` するとBackground起動中も「使用中」に当たります  

ということです。  

**その時の使用中の間のみ許可する** ため、「使用中」状態が終了したタイミングで **「未設定」状態に戻ります** 。  
実際に、 `CLAuthorizationStatus` も `.notDetermined` に戻ります。  

一度「未設定」状態に戻ると、改めて位置情報の利用許可を求めなければ、位置情報サービスを利用することができません。  

この「改めて位置情報の利用許可を求めるタイミング」ですが、アプリのフローに密接に結びつくため、非常に重要です。  
全く関係のないタイミングで位置情報の利用許可を求めると、ユーザに不信感を持たれたり、煩わしさから離れていってしまう可能性もあるでしょう。  

では、最適なタイミングとはいつになるのでしょうか？  
それは、ユーザが再び「位置情報を利用する機能を使いたい」と思ったタイミングになります。  

例えば、レストランの検索アプリの場合、「検索開始ボタンをタップしたタイミング」となるでしょう。  

大事なことは、  
**『適切なタイミングで、ユーザが位置情報サービスを承諾するまで問いかける導線を用意しておく』**  
ということになります。  

因みに、「一時的に許可」を選択した後で、『設定アプリ > プライバシー > 位置情報サービス』を見ると、 **『次回確認』** が設定されています。  

![設定 > プライバシー > 位置情報サービスの設定状態](/images/ios13-core-location_6.jpg)  

### まとめ
さて如何でしたでしょうか。  
iOS13で仕様がシンプルになったからこそ、「位置情報サービスの利用許可を求める最適なタイミング」をしっかりと考える必要が出てきました。  
また、この仕様変更は、アプリ提供元だけでなくユーザにとっても、わかりやすい仕様になったと言えるのではないでしょうか。  

昔からある機能で慣れ親しみがあるものの、その機能全てがユーザに受け入れられているわけではありません。  
少なからず、デメリットを感じてしまう側面もあることでしょう。  
でも、だからこそ、継続して安心安全な機能提供をブラッシュアップしていくことは非常に大切なのでしょう。  

我々、アプリ開発者もそういったことを十分に理解した上で、より良い形でユーザに価値を提供し続けていけると良いですね。  

といったところで本日はここまで。  

### 参考

* [WWDC2019 - What's New in Core Location](https://developer.apple.com/videos/play/wwdc2019/705/)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
