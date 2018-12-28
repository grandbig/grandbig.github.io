---
layout: post
title: "RxDataSourcesを使ってみよう"
date: 2018-12-28 18:41
comments: true
categories: ios swift rx
---

### はじめに
さて今回は [RxDataSources](https://github.com/RxSwiftCommunity/RxDataSources) の使い方について見ていきたいと思います。  

`RxDataSources` を利用することで、  
`Cell` の選択/移動/削除などの扱いが書きやすくなるとのことのようです。  

では早速見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### 今回利用するライブラリをインストール
まずは、今回の紹介サンプルで利用するライブラリのインストールから始めましょう。  
`CocoaPods` を使いますので、下記のように `Podfile` を作成します。  

```objective-c
# Podfile
platform :ios, "11.0"
use_frameworks!

target "RxDataSourcesSample" do
	pod 'RxSwift',    '~> 4.0'
  pod 'RxCocoa',    '~> 4.0'
  pod 'RxDataSources', '~> 3.0'
end

target "RxDataSourcesSampleTests" do
	pod 'RealmSwift'
	pod 'RxBlocking', '~> 4.0'
  pod 'RxTest',     '~> 4.0'
end

target "RxDataSourcesSampleUITests" do
	pod 'RealmSwift'
	pod 'RxBlocking', '~> 4.0'
  pod 'RxTest',     '~> 4.0'
end
```

### RxDataSourcesを利用したサンプル
準備ができたので、実際に `ViewController` にサンプルを書いてみましょう。  

#### プロジェクト構成
因みに、今回のプロジェクト構成は下記のようになっています。  

```objective-c
RxDataSourcesSample
  ├── Model
  │    └── SectionModel
  ├── AppDelegate.swift
  ├── ViewController.swift
  ├── Main.storyboard
  ...
```

#### SectionModelの実装
Model配下に配置した `SectionModel` を実装します。  
これは `RxDataSources` を利用するにあたって根幹をなす `Model` となるため非常に重要です。

```objective-c
// SectionModel.swift
import RxDataSources

struct SectionModel {
    var items: [Item]
}
extension SectionModel: SectionModelType {
    typealias Item = (String, Int)

    init(original: SectionModel, items: [Item]) {
        self = original
        self.items = items
    }
}
```

今回のサンプルでは `Header` は特にセットしないため、 `cell` 内に表示するデータを持つために `items` のみ定義します。  
`SectionModel` は `struct` (構造体)で定義をし、`SectionModelType` を継承させます。  

`SectionModelType` の中身を覗いてみると非常にシンプルな作りになっています。  

```objective-c
import Foundation

public protocol SectionModelType {
    associatedtype Item

    var items: [Item] { get }

    init(original: Self, items: [Item])
}
```

### Storyboardの実装
`Main.storyboard` は下図のように実装します。  

![Main.storyboardの実装](/images/rxdatasources.png)  

### ViewControllerの実装
準備が整ったので `ViewController` を実装していきましょう。  

```objective-c
import UIKit
import RxSwift
import RxCocoa
import RxDataSources

class ViewController: UIViewController {

    // MARK: - IBOutlets
    @IBOutlet private weak var tableView: UITableView!

    // MARK: - Properties
    private let disposeBag = DisposeBag()

    // dataSourceをRxDataSourcesを利用して定義する
    private var dataSource: RxTableViewSectionedReloadDataSource<SectionModel>!

    // cellに設定するデータを保持する
    private var sectionModels: [SectionModel]!

    // cellに表示するデータの変更を検知して、dataSourceに知らせる
    private var dataRelay = BehaviorRelay<[SectionModel]>(value: [])

    // MARK: - Lifecycle methods
    override func viewDidLoad() {
        super.viewDidLoad()

        // Cellに設定するデータを格納
        sectionModels = [SectionModel(items: [("test1", 1), ("test2", 2), ("test3", 3)])]

        // RxDataSourcesを利用してCellを描画
        dataSource = RxTableViewSectionedReloadDataSource<SectionModel>(
            configureCell: { _, tableView, indexPath, item in
                // 引数名通り、与えられたデータを利用してcellを生成する
                let cell = tableView.dequeueReusableCell(withIdentifier: "Cell",
                                                         for: IndexPath(row: indexPath.row, section: 0))
                cell.textLabel?.text = item.0
                cell.accessoryType = .disclosureIndicator

                return cell
        }, canEditRowAtIndexPath: { _, _ in
            // この引数を設定しないと、Cellの削除アクションができない
            return true
        })

        // dataRelayの変更をキャッチしてdataSourceにデータを流す
        dataRelay.asObservable()
            .bind(to: tableView.rx.items(dataSource: dataSource))
            .disposed(by: disposeBag)

        // Cellを削除した場合にバインディングされる処理
        tableView.rx.itemDeleted
            .subscribe(onNext: { [weak self] indexPath in
                guard let strongSelf = self, let sectionModel = strongSelf.sectionModels.first else { return }
                var items = sectionModel.items
                items.remove(at: indexPath.row)

                // dataRelayにデータを流し込む
                strongSelf.dataRelay.accept([SectionModel(items: items)])
            })
            .disposed(by: disposeBag)

        // 初期表示用のデータフェッチ
        fetch()
    }
}

// MARK: - Private methods
extension ViewController {

    // 初期表示用のデータフェッチする処理
    private func fetch() {
        // sectionModelsを利用して
        Observable.just(sectionModels)
            .subscribe(onNext: { [weak self] _ in
                guard let strongSelf = self else { return }

                // dataRelayにデータを流し込む
                strongSelf.dataRelay.accept(strongSelf.sectionModels)
            })
            .disposed(by: disposeBag)
    }
}
```

因みに `Cell` を削除した場合に `deleteRow` を実行する必要はありません。  
理由は、 `RxTableViewSectionedReloadDataSource` を利用すると `reloadData` が実行されるようになっているためです。  

```objective-c
// RxTableViewSectionedReloadDataSource.swift

#if os(iOS) || os(tvOS)
import Foundation
import UIKit
#if !RX_NO_MODULE
import RxSwift
import RxCocoa
#endif
import Differentiator

open class RxTableViewSectionedReloadDataSource<S: SectionModelType>
    : TableViewSectionedDataSource<S>
    , RxTableViewDataSourceType {
    public typealias Element = [S]

    open func tableView(_ tableView: UITableView, observedEvent: Event<Element>) {
        Binder(self) { dataSource, element in
            #if DEBUG
                self._dataSourceBound = true
            #endif
            dataSource.setSections(element)
            tableView.reloadData()  --> reloadDataを実行するようになっている
        }.on(observedEvent)
    }
}
#endif
```

### まとめ
さて如何でしたでしょうか？  
書き方さえ慣れてしまえば案外簡単に利用できそうですよね。  

ということで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
