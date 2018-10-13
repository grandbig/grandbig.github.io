---
layout: post
title: "下からニュッと出るPickerを作ろう！"
date: 2018-10-13 12:45
comments: true
categories: ios swift
---

### はじめに
今日は様々記事で既に書かれている議題をあえて使おうと思います。  
というのもほんのちょっとしたミスで非常にハマってしまった備忘録を残すためです。  

では早速見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### 作成手順

① `New File...` > `User Interface` > `View` を選択し、 `xib` ファイルを作成しましょう  

![xibファイルの作成](/images/popuppickerview_1.png)  

② `xib` 上で `UIPickerView` と `UIToolBar` を追加しましょう  

![UIPickerViewとUIToolBarの追加](/images/popuppickerview_2.png)  

③ `xib` ファイルに対応する `swift` ファイルを作成しましょう  
`New File...` > `Source` > `Swift File` を選択します。  

![swiftファイルの作成](/images/popuppickerview_3.png)  

このタイミングで最低限、以下を定義しておきましょう。  

```objective-c
import UIKit

class PickerView: UIView {

}
```

④ `xib` ファイルと `swift` ファイルの対応付け  

下図のように、 `Placeholders` > `File's Owner` を選択して、  
右メニューの `Show the Identity inspector` から `Custom Class` を設定します。  

![xibファイルとswiftファイルの対応付け](/images/popuppickerview_4.png)  

⑤ `swift` ファイルに具体的な実装を書きましょう  

始めに結論を書いてしますと以下の通りです。  

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
    private var selectedRow: Int = 0

    // MARK: - Initial Methods
    required init(frame: CGRect = CGRect(x: 0, y: screenHeight, width: screenWidth, height: defaultPickerHeight),
                  selectItems: [String]) {
        var frame = frame
        // 説明(1)
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

    // 説明(2)
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
        delegate?.selectedItem(index: selectedRow)
        hiddenPickerView()
    }
}

/// MARK: - UIPickerViewDelegate
extension PickerView: UIPickerViewDelegate {

    public func pickerView(_ pickerView: UIPickerView, titleForRow row: Int, forComponent component: Int) -> String? {
        return selectItems[row]
    }

    public func pickerView(_ pickerView: UIPickerView, didSelectRow row: Int, inComponent component: Int) {
        selectedRow = row
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

// 説明(3)
/// MARK: - PickerViewDelegate
public protocol PickerViewDelegate: class {
    func selectedItem(index: Int)
    func closePickerView()
}
```

一部詳細を説明します。  

#### 説明(1)
昨年から `iPhoneX` が発売されましたので、 `UIPickerView` の高さを調整する必要があります。  
今のところ、 `safeAreaInsets.top` が存在するのは、  
`iPhoneX / XS / XS MAX / XR` といったホームボタンのない全面ディスプレイのみですので、  
`safeAreaInsets.top > CGFloat(0.0)` で判定できるでしょう。  

#### 説明(2)
下からニュッと出たり、下にニュッと引っ込んだりする動きは単に `UIView` の位置を `animate` メソッドで動かしているだけです。  

#### 説明(3)
`UIToolBar` 上のボタンクリックで `UIPickerView` の挙動を操作したい場合があるでしょう。  
そこで独自の `Delegate Methods` を用意しています。  
この呼出は `IBAction` でハンドリングしている `cancelSelection` と `doneSelection` に紐づけています。    

### 筆者が躓いた点について備忘録
筆者が本実装をしている際に、どうしても `UIPickerViewDelegate` のメソッドが呼び出されないという現象がありました。  
原因は超絶基本的なことだったのですが、備忘録として残そうと思います。  

まずは、誤っているソースコードから  

```objective-c
/// MARK: - UIPickerViewDelegate
extension PickerView: UIPickerViewDelegate {

    private func pickerView(_ pickerView: UIPickerView, titleForRow row: Int, forComponent component: Int) -> String? {
        return selectItems[row]
    }

    private func pickerView(_ pickerView: UIPickerView, didSelectRow row: Int, inComponent component: Int) {
        selectedRow = row
    }
}
```

そして、正しいソースコードは  

```objective-c
/// MARK: - UIPickerViewDelegate
extension PickerView: UIPickerViewDelegate {

    public func pickerView(_ pickerView: UIPickerView, titleForRow row: Int, forComponent component: Int) -> String? {
        return selectItems[row]
    }

    public func pickerView(_ pickerView: UIPickerView, didSelectRow row: Int, inComponent component: Int) {
        selectedRow = row
    }
}
```

上記2つの違いが何かと言うと、  
メソッドを `private` で定義しているか `public` で定義しているかだけです。  
`private` にしていたためスコープ外となってしまい呼び出せていなかったのでした...  
何という凡ミス...これで3時間ほど持っていかれました笑  

### まとめ
さて今回は筆者の備忘録的な記事になりましたが、  
`UIPickerView` は相変わらずニュッと下から出る挙動はデフォルトで提供してくれていないため、参考になることでしょう。  

と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
