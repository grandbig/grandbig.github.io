---
layout: post
title: "カスタムDelegateのRx対応"
date: 2018-10-14 22:52
comments: true
categories: ios swift rx
---

### はじめに
先日、[GeolocationSampleから学ぶdelegateのRx対応](https://grandbig.github.io/blog/2018/10/06/rx-delegate/)を紹介しました。  
今回は[下からニュッと出るPickerを作ろう！](https://grandbig.github.io/blog/2018/10/13/popuppickerview/)で作成した `PickerView` に実装されている `Delegate` を `Rx` 対応させたいと思います。

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### PickerViewクラスの確認
まずは、元となる `PickerView` クラスを提示します。  

```objective-c
import Foundation
import UIKit

/// ピッカービュー
public class PickerView: UIView {

    // MARK: - IBOutlets
    @IBOutlet weak var toolBar: UIToolbar!
    @IBOutlet weak var picker: UIPickerView!

    // MARK: - Static Properties
    static private let screenWidth = UIScreen.main.bounds.size.width
    static private let screenHeight = UIScreen.main.bounds.size.height
    static private let defaultPickerHeight: CGFloat = 260.0
    static private let duration = 0.2

    // MARK: - Properties
    public weak var delegate: PickerViewDelegate?
    private var selectItems = [String]()
    private var selectedRowIndex: Int = 0

    // MARK: - Initial Methods
    required init(frame: CGRect = CGRect(x: 0, y: screenHeight, width: screenWidth, height: defaultPickerHeight),
                  selectItems: [String]) {
        var frame = frame
        if let safeAreaTopInsets = UIApplication.shared.keyWindow?.safeAreaInsets.top, safeAreaTopInsets > CGFloat(0.0) {
            // iPhoneX , XS, XS MAX, XRの場合はUIPickerViewの高さを調整する
            frame = CGRect(x: 0, y: frame.origin.y, width: frame.size.width, height: (frame.size.height + 100.0))
        }
        super.init(frame: frame)
        self.selectItems = selectItems
        self.xibViewSet()
    }

    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)!
        self.xibViewSet()
    }

    internal func xibViewSet() {
        if let view = R.nib.pickerView.firstView(owner: self) {
            view.frame = self.bounds
            self.addSubview(view)

            picker.delegate = self
            picker.dataSource = self
            picker.showsSelectionIndicator = true
        }
    }

    // MARK: - Picker Move Function
    // PickerViewを表示する
    func showPickerView() {
        let pickerViewWidth = self.frame.size.width
        let pickerViewHeight = self.frame.size.height
        let pickerViewYPosition = PickerView.screenHeight - pickerViewHeight
        UIView.animate(withDuration: PickerView.duration) {
            self.frame = CGRect.init(x: 0, y: pickerViewYPosition, width: pickerViewWidth, height: pickerViewHeight)
        }
    }

    // PickerViewを非表示にする
    func hiddenPickerView() {
        let pickerViewWidth = self.frame.size.width
        let pickerViewHeight = self.frame.size.height
        UIView.animate(withDuration: PickerView.duration) {
            self.frame = CGRect.init(x: 0, y: PickerView.screenHeight, width: pickerViewWidth, height: pickerViewHeight)
        }
    }

    // MARK: - IBActions
    @IBAction func cancelSelection(_ sender: Any) {
        delegate?.closePickerView()
        hiddenPickerView()
    }

    @IBAction func doneSelection(_ sender: Any) {
        delegate?.selectedItem(index: selectedRowIndex, title: selectItems[selectedRowIndex])
        hiddenPickerView()
    }
}

/// MARK: - UIPickerViewDelegate
extension PickerView: UIPickerViewDelegate {

    public func pickerView(_ pickerView: UIPickerView, titleForRow row: Int, forComponent component: Int) -> String? {
        return selectItems[row]
    }

    public func pickerView(_ pickerView: UIPickerView, didSelectRow row: Int, inComponent component: Int) {
        selectedRowIndex = row
    }
}

/// MARK: - UIPickerViewDataSource
extension PickerView: UIPickerViewDataSource {

    public func numberOfComponents(in pickerView: UIPickerView) -> Int {
        return 1
    }

    public func pickerView(_ pickerView: UIPickerView, numberOfRowsInComponent component: Int) -> Int {
        return selectItems.count
    }
}

/// MARK: - PickerViewDelegate
@objc
public protocol PickerViewDelegate: class {
    func selectedItem(index: Int, title: String)
    func closePickerView()
}
```

では早速 `Rx` 対応させていきましょう。  

### DelegateProxyとDelegateProxyTypeへの対応
基本的には、 `CLLocationManagerDelegate` と同じです。  
`DelegateProxy` と `DelegateProxyType` を継承したクラスを実装します。  

```objective-c
import Foundation
import RxSwift
import RxCocoa

// 説明(1)
extension PickerView: HasDelegate {
    public typealias Delegate = PickerViewDelegate
}

public class RxPickerViewDelegateProxy: DelegateProxy<PickerView, PickerViewDelegate>,
    DelegateProxyType,
    PickerViewDelegate {

    public init(pickerView: PickerView) {
        super.init(parentObject: pickerView, delegateProxy: RxPickerViewDelegateProxy.self)
    }

    // 説明(2)
    public static func registerKnownImplementations() {
        self.register { RxPickerViewDelegateProxy(pickerView: $0) }
    }

    // 説明(3)
    internal lazy var selectedItemSubject = PublishSubject<(Int, String)>()
    internal lazy var closePickerViewSubject = PublishSubject<Void>()

    // 説明(4)
    public func selectedItem(index: Int, title: String) {
        selectedItemSubject.onNext((index, title))
    }

    public func closePickerView() {
        closePickerViewSubject.onNext(Void())
    }

    // 説明(5)
    deinit {
        self.selectedItemSubject.on(.completed)
        self.closePickerViewSubject.on(.completed)
    }
}
```

細かく見ていきましょう。  

#### 説明(1)
`currentDelegate` および `setCurrentDelegate` に対応する代わりに、 `HasDelegate` を継承させましょう。  

#### 説明(2)
自身で定義した `DelegateProxy` の継承クラスを登録するために、 `registerKnownImplementations` 内で `DelegateProxySubclass.register()` を実行します。

#### 説明(3)
`delegate` メソッドが呼び出されて処理が実行されたことを `Subscriber` に伝えるために、 `PublishSubject` 型のプロパティを用意します。  

#### 説明(4)
`PickerViewDelegate` の `selectedItem(index:title:)` と `closePickerView()` メソッドは必須メソッドです。  
`RxPickerViewDelegateProxy` はもちろん `PickerViewDelegate` も継承しますので、上記2つのメソッドを定義する必要があります。  

これが呼び出されたタイミングで `Subscriber` に伝えるために、 `PublishSubject.onNext(element:)` を実行します。  

#### 説明(5)
`deinit` が呼ばれるタイミングで、初期化したオブジェクトが破棄されるので、  
`PublishSubject` からイベント送信完了を知らせるように実装しましょう。  

### ReactiveへのPickerViewの適応
これも `CLLocationManager` と適応方法は同じです。  
まずは、全体像から...  

```objective-c
// PickerView+Rx.swift
import Foundation
import RxSwift
import RxCocoa

extension Reactive where Base: PickerView {

    // 説明(1)
    public var delegate: DelegateProxy<PickerView, PickerViewDelegate> {
        return RxPickerViewDelegateProxy.proxy(for: base)
    }

    // 説明(2)
    public var selectedItem: Observable<(Int, String)> {
        return RxPickerViewDelegateProxy.proxy(for: base).selectedItemSubject.asObservable()
    }

    public var closePickerView: Observable<Void> {
        return RxPickerViewDelegateProxy.proxy(for: base).closePickerViewSubject.asObservable()
    }
}
```

1つずつ説明します。  

#### 説明(1)
`delegate` を `DelegateProxy` 型として定義します。  
`DelegateProxy` の取得は `DelegateProxyType` プロトコルの `proxy` メソッドを利用します。  

#### 説明(2)
各 `delegate` メソッドが実行されたことを補足(監視)するために `Observable` 型の `selectedItem` と `closePickerView` を用意します。  

### 利用方法
では、早速利用してみましょう。  

```objective-c
// ViewController.swift

private let disposeBag = DisposeBag()

private func bind() {
  pickerView?.rx.selectedItem
      .asObservable()
      .subscribe({ [weak self] event in
          // Subscriberとして補足した情報を取得
          guard let strongSelf = self else { return }
          guard let index = event.element?.0 else { return }
          guard let title = event.element?.1 else { return }

          Observable.just(index, title)
              // ViewModelにsampleActionが定義されているとします
              .bind(to: strongSelf.viewModel.sampleAction)
              .disposed(by: strongSelf.disposeBag)
              })
              .disposed(by: disposeBag)
}
```

上記のような形で `ViewController` にて `Subscriber` としてアクションを補足し、 `ViewModel` に伝えることができるでしょう。  

### まとめ
さて、如何でしたでしょうか？  
1つ1つの意味を理解することはもちろん大切ですが、  
何だか型にはまって書き方を覚えれば、自身で `Rx` 対応が簡単にできる気がしてきますね。  

`Rx` の癖が強いが故に、慣れれば利用しやすいということなのでしょう。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
