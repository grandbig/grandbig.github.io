---
layout: post
title: "This is a feature to warn you that there is already a delegate の対応 ~ RxSwiftでUITableViewのリロード時にクラッシュする問題にぶつかった ~"
date: 2018-12-30 04:11
comments: true
categories: rx ios swift
---

### はじめに
今回は表題にあるクラッシュ問題についてのメモです。  
単純な話だったけど、しばらくハマってました笑  

取り組んでいた内容としてはシンプルで、  

* `RxSwift` を利用していた  
* `RxDataSources` を利用しようとした  
* `UITableView` の `Cell` 削除アクションを `Rx` っぽく書きたかった  

というものです。  

`UITableView` の初期描画はうまくいくものの、 `Cell` 削除アクションを実行するとクラッシュしていました。  

では早速内容について見ていきましょう。   

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### クラッシュの内容
どんなクラッシュが発生していたかというと  

```objective-c
This is a feature to warn you that there is already a delegate (or data source) set somewhere previously.
The action you are trying to perform will clear that delegate (data source) and that means that some of your features
that depend on that delegate (data source) being set will likely stop working.\n
```

といったものが出ていました。  

この内容でググってみると...  

* [Maybe delegate was already set in `xib` or `storyboard` and now it's being overwritten in code.](https://github.com/ReactiveX/RxSwift/issues/675)  
* [How to change datasource and rebind to tableview](https://github.com/RxSwiftCommunity/RxDataSources/issues/185)  
* [Assertion failure in DelegateProxyType](https://github.com/ReactiveX/RxSwift/issues/706)  

といった形でたびたび本家の `RxSwift` で意見交換されていました。  

上記内で言われていることは、  

```
tableView.delegate = nil
tableView.dataSource = nil
```

を書けば解決するよって話だったりしました。  
筆者的には、 `xib` で `delegate` や `dataSource` の設定などしていなかったので、半信半疑ながら上記をコードに記載して試していました。  

が解決されず...  

### 解決方法
では、一体どうやって解決したかというと、  
クラッシュの内容とは全く関係のない部分の話でした...  

筆者は `MVVM` アーキテクチャでプロジェクトを構成しており、  
下記のように `ViewModel` を定義していました。  

```objective-c
// ViewModel.swift
import Foundation
import RxSwift
import RxCocoa

final class MainViewModel: Injectable {

    struct Dependency {
    }

    // MARK: - Properties
    private let disposeBag = DisposeBag()
    private var sectionModels: [SectionModel]!

    // MARK: PublishRelays
    let requestDeleteRecordStream = PublishRelay<IndexPath>()

    // MARK: BehaviorRelays
    var dataRelay = BehaviorRelay<[SectionModel]>(value: [])

    // MARK: Initial method
    init(with dependency: Dependency) {

        sectionModels = [SectionModel(items: [("test1", 1), ("test2", 2), ("test3", 3)])]

        Observable.deferred {() -> Observable<[SectionModel]> in
            return Observable.just(self.sectionModels)
            }
            .bind(to: dataRelay)
            .disposed(by: disposeBag)

        requestDeleteRecordStream
            .subscribe(onNext: { [weak self] indexPath in
                guard let strongSelf = self, let sectionModel = strongSelf.sectionModels.first else { return }
                var items = sectionModel.items
                items.remove(at: indexPath.row)
                strongSelf.sectionModels = [SectionModel(items: items)]
                strongSelf.dataRelay.accept(strongSelf.sectionModels)
            })
            .disposed(by: disposeBag)
    }
}
```

続いて、 `View` の方では下記のように、 `Cell` が削除された場合を捕捉するようにしていました。  

```objective-c
// View
import UIKit
import RxSwift
import RxCocoa
import RxDataSources

class MainViewController: UIViewController, Injectable {
    typealias Dependency = MainViewModel

    @IBOutlet private weak var tableView: UITableView!

    private let disposeBag = DisposeBag()
    private var dataSource: RxTableViewSectionedReloadDataSource<SectionModel>!
    private let viewModel: MainViewModel

    required init(with dependency: Dependency) {
        viewModel = dependency
        super.init(nibName: nil, bundle: nil)
    }

    @available(*, unavailable)
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view.
        tableView.register(CustomTableViewCell.self, forCellReuseIdentifier: "Cell")

        dataSource = RxTableViewSectionedReloadDataSource<SectionModel>(
            configureCell: { _, tableView, indexPath, item in
                let cell = tableView.dequeueReusableCell(withIdentifier: "Cell",
                                                         for: IndexPath(row: indexPath.row, section: 0))
                cell.textLabel?.text = item.0
                cell.accessoryType = .disclosureIndicator

                return cell
        }, canEditRowAtIndexPath: { _, _ in
            return true
        })

        //////// ↓問題はココ /////////////////////////////////////////////////////////////////////
        viewModel.dataRelay
            .subscribe(onNext: { [weak self] records in
                guard let strongSelf = self, let items = records.first?.items else { return }
                let data = [SectionModel(items: items)]
                Observable.just(data)
                    .bind(to: strongSelf.tableView.rx.items(dataSource: strongSelf.dataSource))
                    .disposed(by: strongSelf.disposeBag)
            }.disposed(by: strongSelf.disposeBag)
        })
        //////// ↑問題はココ /////////////////////////////////////////////////////////////////////

        tableView.rx.itemDeleted
            .subscribe(onNext: { [weak self] indexPath in
                guard let strongSelf = self else { return }
                Observable.just(indexPath)
                    .bind(to: strongSelf.viewModel.requestDeleteRecordStream)
                    .disposed(by: strongSelf.disposeBag)
            })
            .disposed(by: disposeBag)
    }
}
```

上記の **問題はココ** と書かれているところが間違っていました。  
`ViewModel` 側で既に `SectionModel` としてデータを作成しているので、  
`ViewController` 側では単に `tableView.rx.items` に流せば良かったのです。  

それを改めて `SectionModel` の形に整形し直そうとしてしまっていました。  
そして再度 `Observable` を生成して `tableView.rx.items` に流し込んでいました。  

この問題の部分を下記のように修正したところクラッシュすることがなくなりました。  

```objective-c
viewModel.dataRelay
    .asObservable()
    .bind(to: tableView.rx.items(dataSource: dataSource))
    .disposed(by: disposeBag)
```

### まとめ
クラッシュ内容にだいぶ惑わされましたが、  
どうしてもわからない時は簡単なサンプルを作って試してみるのが良いなと改めて思いました。  

ということで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
