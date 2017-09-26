---
layout: post
title: "Clean Swiftを勉強してみよう！(1)"
date: 2017-09-25 23:42
comments: true
categories: ios swift
---

### はじめに
本日は[Clean Swift](https://clean-swift.com/)について書いていきたいと思います。  

#### Clean Swiftとは
Clean Swiftは簡単に言うと『[Clean Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html)のSwift版』です。    
参考までにClean Architectureの有名な図を掲載します。  
![Clean Architecture](/images/clean-swift_1.jpg)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

Clean Swiftアーキテクチャを採用することで受けられる恩恵として下記が考えられます。  

* 各種コンポーネントの責務を細分化することで、Massive ViewControllerの解消に繋がる  
* データの方向性が一方向になるため、各種コンポーネントの相互依存性が減り、TDD開発が進めやすくなる  
* 各種コンポーネントの責務がはっきりしているため、チーム開発する際に、実装が平準化される  

#### コンポーネントの関係性
各種コンポーネントの関係性を表した個人的に最もわかりやすいと感じた全体像が下図になります。  
[swifting.io](https://swifting.io/blog/2016/09/07/architecture-wars-a-new-hope/)から抜粋させて頂きました。  
![Clean Swift Architectureの図](/images/clean-swift_2.png)  

この関係性を説明するにあたって、各種コンポーネントの責務を理解しておく必要があるのでそれぞれ見ていきましょう。  

#### View
特に他のアーキテクチャと大きく違わない認識です。  

**責務：**  
・iOSアプリの見た目を表現する  

#### ViewController
`Massive ViewController` になりがちな部分ですが、Clean Swiftでの責務は以下になります。  

**責務：**  
① `Interactor` に具体的な処理内容(表示ロジック)を問い合わせる  
② `Presenter` からの指示を受けて、最適な `View` を描画する  
③ `Router` に画面遷移を依頼する  

具体例を下記に記します。  

**【例題】**  
画面の初期表示時に何らかのデータを取得して表示する  

```objective-c
import UIKit

protocol SampleViewDisplayLogic: class {
  func displaySomething(viewModel: SampleView.Something.ViewModel)
  func transitionToSomeWhere(viewModel: SampleView.Sometime.ViewModel)
}

class SampleViewController: UIViewController, SampleViewDisplayLogic {
  var interactor: SampleViewBusinessLogic?
  var router: (NSObjectProtocol & SampleViewRoutingLogic & SampleViewDataPassing)?

  // MARK: Object lifecycle
  override init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: Bundle?) {
    super.init(nibName: nibNameOrNil, bundle: nibBundleOrNil)
    setup()
  }

  required init?(coder aDecoder: NSCoder) {
    super.init(coder: aDecoder)
    setup()
  }

  // MARK: Setup
  private func setup() {
    let viewController = self
    let interactor = SampleViewInteractor()
    let presenter = SampleViewPresenter()
    let router = SampleViewRouter()
    viewController.interactor = interactor
    viewController.router = router
    interactor.presenter = presenter
    presenter.viewController = viewController
    router.viewController = viewController
    router.dataStore = interactor
  }

  // MARK: Routing
  override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    if let scene = segue.identifier {
      let selector = NSSelectorFromString("routeTo\(scene)WithSegue:")
      if let router = router, router.responds(to: selector) {
        router.perform(selector, with: segue)
      }
    }
  }

  // MARK: View lifecycle
  override func viewDidLoad() {
    super.viewDidLoad()

    fetchSomethingOnLoad()
  }

  // ① Interactorに具体的な処理内容を問い合わせる
  func fetchSomethingOnLoad() {
    let request = SampleView.Something.Request()
    interactor?.fetchSomething(request: request)
  }

  // ② Presenterからの指示を受けてViewを描画する
  func displaySomething(viewModel: SampleView.Something.ViewModel) {
    // do something
  }

  // ③ Routerに画面遷移を依頼する
  func transitionToSomeWhere(viewModel: SampleView.Sometime.ViewModel) {
    // 画面遷移
    router?.routeToSomeWhere(segue: nil)
  }
}
```

また、ユーザによるアクション起因の場合は下記のようにするだけです。  

```objective-c
@IBAction func tapSomeAction(_ sender: Any) {
  ① Interactorに具体的な処理内容を問い合わせる
  let request = SampleView.Sometime.Request()
  interactor?.fetchSometime(request: request)
}
```

`Presenter` からの指示を受けて、 `ViewController` は描画処理を実行するため、見た目の整形などの描画処理自体は `ViewController` 内に書きます。  

例えば、  

・正方形の `UIView` を角丸にする/背景色を変更する/非表示にする etc  
・マップにマーカを配置する/図形を描画する etc  

#### Interactor
`ViewController` から依頼を受け、 `Interactor` は下記を実施する責務を持っています。  

**責務：**  
① `Worker` と `Presenter` を仲介する  
② どんな条件で、 `Worker` に何の処理を依頼するのかハンドリングする  
③ `Worker` 経由で取得したレスポンスを `Presenter` に渡す  

```objective-c
import UIKit

protocol SampleViewBusinessLogic {
    func fetchSomething(request: SampleView.Something.Request)
    func fetchSometime(request: SampleView.Sometime.Request)
}

protocol SampleViewDataStore {
  // 画面遷移時にパラメータを受け取れるように定義
  var something: String { get set }
}

class SampleViewInteractor: SampleViewBusinessLogic, SampleViewDataStore {
    var presenter: SampleViewPresentationLogic?
    var worker = SampleViewWorker?
    var something: String!

    func fetchSomething(request: SampleView.Something.Request) {
      // ① WorkerとPresenterを仲介する
      worker.fetch() { (object) in
        // ③ Worker経由で取得したレスポンスをPresenterに渡す  
        let response = SampleView.Something.Response(object: object)
        self.presenter?.presentSomething(response: response)
      }
    }

    func fetchSometime(request: SampleView.Sometime.Request) {
      ② どんな条件で、Workerに何の処理を依頼するのかハンドリングする
      if request.time > Date() {
        let response = SampleView.Sometime.Response(future: true)
        presenter?.presentSometime(response: response)

        return
      }
      let response = SampleView.Sometime.Response(future: false)
      presenter?.presentSometime(response: response)
    }

    func fetchSomeWhat(request: SampleView.SomeWhat.Request) {
      // 画面遷移時に渡されたパラメータを利用した描画を実施したい場合
      let response = SampleView.Something.Response(object: something)
      self.presenter?.presentSomething(response: response)
    }
}
```

#### Worker
`Interactor` から受けた依頼を実行します。  

**責務：**  
・ `API` 処理や `Core Data` / `Realm` などのアプリ内ローカルデータの処理をハンドリングする  
・ 取得データを `Model.Response` 形式で返却する  
・ 成功/失敗レスポンスをハンドリングする  

```objective-c
import UIKit

typealias responseHandler = (_ response: SampleView.Something.Response) ->()

class SampleViewWorker {

    func fetch(completion: @escaping (responseHandler) {
      // APIリクエストまたはローカルDBへのアクセスを実行してデータを取得
      completion()
    }
}
```

#### Presenter
`Interactor` から `Worker` 経由で取得したレスポンスを受け取った後に、 `Presenter` は下記を実行することを責務とします。  

**責務：**  
・ 受け取ったレスポンスを元に最適な表示(成功/失敗などの表示)になるようハンドリングする  
・ 受け取ったレスポンスを `Model.ViewModel` 形式に変換する  
・ `ViewController` に `Model.ViewModel` を渡して描画を依頼する  

#### Model
Clean Swiftアーキテクチャの肝といっても過言ではないのが `Model` です。  

**責務：**  
・ 各種コンポーネントを切り離し、各種コンポーネント間のやり取りに利用される  
・ `Request` / `Response` / `ViewModel` の3つの構造体を持つ  

**3つの構造体の説明：**  
・ `Request`  
　　・ ユーザの操作をInputパラメータとして内包したデータ形式  
　　・ `ViewController` から `Interactor` に渡される  
・ `Response`  
　　・ `Worker` 処理結果を内包しているデータ形式  
　　・ `Interactor` から `Presenter` に渡される  
・ `ViewModel`  
　　・ `ViewController` での描画に即したデータ形式  
　　・ `Presenter` から `ViewController` に渡される  


### まとめ


参考URL:  

* [Clean Swift公式ページ](https://clean-swift.com/clean-swift-ios-architecture/)  
* [Introducing Clean Swift Architecture (VIP)](https://hackernoon.com/introducing-clean-swift-architecture-vip-770a639ad7bf)  
* [swifting.io: #24 Architecture Wars – A New Hope](https://swifting.io/blog/2016/09/07/architecture-wars-a-new-hope/)  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
