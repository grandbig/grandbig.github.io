---
layout: post
title: "iOSエンジニアが苦しんだReactアニメーション"
date: 2017-02-04 21:54
comments: true
categories: FE, javascript, react, redux, web
---

###はじめに
少し期間が空いてしまいましたが、ブログを更新します。  
今回は理解できていれば簡単なのに、なかなか実装できなかったReactアニメーションについてです。  
これが何でかすご〜く苦労したんですね...  
筆者自身が詰まったところから紐解く形で解説を載せていきたいと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

###Counterサンプルにアニメーションを追加しよう！
まずはネタとして[前回](http://grandbig.github.io/blog/2017/01/02/redux-base-4/)まで基礎を学ぶのに利用した`Counter`サンプルを利用します。  
元々の`Counter`サンプルとは以下のものになります。  

![Counter Example画面](/images/redux_base_1.png)  

実装されている機能としては下記の4つになります。  

* 「+」ボタンを選択するとClick数が+1される  
* 「-」ボタンを選択するとClick数が-1される  
* 「Increment if odd」ボタンを選択するとClick数が奇数のときのみ+1される  
* 「Increment async」ボタンを選択すると1秒後にClick数が+1される  

これに次の機能を1つ追加します。  

* 「Show Toast」ボタンを選択するとToastが表示される  
* 「Show Toast」ボタンを連打すると追加でToastが表示される  

「**Toast** って何だろう...!?」と思う人もいるかもしれませんので、念のため説明します。  
簡単に言うと、「ユーザにポップアップのような形で知らせるもの」です。  
Androidユーザなら必ず見たことがあるはずです。([Android Developers Toasts](https://developer.android.com/guide/topics/ui/notifiers/toasts.html))

今回目指す完成図は下記になります。  

![機能を追加したCounter Example画面](/images/redux_base_2.png)  

####フォルダ構成
まずは例によってフォルダ構成を見ていきます。  

```javascript
counter
├── public
│     └── index.html
└── src
     ├── index.js
     ├── actions
     │     └── index.js
     ├── reducers
     │     └── index.js
     ├── components
     │     ├── Counter.js
     │     └── Toast
     │           ├── Toast.js
     │           └── Toast.css
     └── node_modules
```

#####Toast表示のActionを追加
さて、まずはToastを表示するための`Action`を追加しましょう。  

```javascript
// actions/index.js
import { createAction } from 'redux-actions';

const INCREMENT = ('INCREMENT');
const DECREMENT = ('DECREMENT');
const SHOW_TOAST = ('SHOW_TOAST');  // 追加

export const increment = createAction(INCREMENT);
export const decrement = createAction(DECREMENT);
export const showToast = createAction(SHOW_TOAST);  // 追加
```

おわかりの通り、`showToast`がそのActionに該当します。  

####Toast表示Action発行後のReducer処理
続いて、`Reducer`の処理を書いていきます。  
機能要件にあった通り、ボタンを連打することで複数のToastを表示します。  
後で詳細を説明しますが、個々のToastを区別する必要があるため、固有値を付与します。  

```javascript
// reducers/index.js
import { handleActions } from 'redux-actions'

const initialState = {
  value: 0,
  num: 0    // 追加 (1)
};

export default handleActions({
  INCREMENT: (state, action) => {
    const newState = {
      ...state,   // 追加 (2)
      value: state.value + 1
    };
    return newState;
  },
  DECREMENT: (state, action) => {
    const newState = {
      ...state,   // 追加 (2)
      value: state.value - 1
    };
    return newState;
  },
  // 以下が追加 (3)
  SHOW_TOAST: (state, action) => {
    const newState = {
      ...state,
      num: state.num + 1
    };
    return newState;
  }
}, initialState)
```

それぞれ説明します。  

追加(1):  
今回は簡単のため各Toastに与えるために固有値を`num`として定義し、初期値を0とします。  
これをボタンタップ時にカウントアップして、その`num`を付与してToastを作成することで個々を区別することができます。  

追加(2):  
他のボタンをタップしたときに`state.num`がリセットされてしまわないように、`...state`を追加して、変化のない値も丸々返すようにしています。  

追加(3):  
Toast表示`Action`が`dispatch`された後に検知して、新たな`num`値を返却するために追加しました。  

####Toast Componentの作成
今回、最も重要な`Component`である`Toast Component`を作成していきましょう。  
Reactで簡単にアニメーションを作成可能な`react-addons-css-transition-group`の`ReactCSSTransitionGroup`を使います。  

```javascript
// ./components/Toast/Toast.js
import React, { Component } from 'react';
import ReactCSSTransitionGroup from 'react-addons-css-transition-group';
import './Toast.css';

class Toast extends Component {

  render() {
    const { num } = this.props;
    const toast = <div key={num} className="toast">Get Wild</div>
    return (
      <div className="toast-group">
        <ReactCSSTransitionGroup
          transitionName="example"
          transitionEnterTimeout={1500}
          transitionLeaveTimeout={1000}
        >
        {toast}
        </ReactCSSTransitionGroup>
      </div>
    )
  }
}

Toast.propTypes = {
  num: React.PropTypes.number.isRequired
}

export default Toast
```

`ReactCSSTransitionGroup Component`の各パラメータは次の通りの意味です。  

* transitionName: 基本となるアニメーション対象要素の名称  
* transitionEnterTimeout: 要素が追加された後のタイムアウト時間  
* transitionLeaveTimeout: 要素が消えた後のタイムアウト時間？  

CSSでもアニメーションを追加するので、`transitionEnterTimeout`や`transitionLeaveTimeout`が不要なんじゃないかと思ったりしたのですが、これらをつけないとエラーが出ます。  

`ReactCSSTransitionGroup`の子要素としてToast表示したいDOM要素を追加します。  
この子要素に`key`パラメータとして`num`を渡しています。  
これによってボタン連打で追加される各DOM要素が固有の要素であることを区別しています。  
`key`の重要性については、[React.jsの地味だけど重要なkeyについて](http://qiita.com/koba04/items/a4d23245d246c53cd49d)を見ることをオススメします。  

`key`に`num`を渡したいので、`Toast`の`propTypes`に`num: React.PropTypes.number.isRequired`と定義しています。  

```css
// ./components/Toast/Toast.css
.toast-group {
  position: fixed;
  top: 10px;
  right: 10px;
}
.toast {
  background-color: #f4df42;
  border-radius: 10px;
  width: 200px;
  height: 35px;
  line-height: 35px;
  margin: 10px;
  text-align: center;
}
.toast:first-child {
  display: none;
}

.example-enter {
  opacity: 0.01;
}

.example-enter.example-enter-active {
  opacity: 1;
  transition: opacity 500ms ease-in;
}

.example-leave {
  opacity: 1;
}

.example-leave.example-leave-active {
  opacity: 0.01;
  transition: opacity 300ms ease-in;
}
```

アニメーションを実行するには`css`ファイルでスタイルをつけることも必要です。  
それぞれ説明します。  

* `.toats-group`  
これは単にToast全体の表示位置を右上に固定したかったため追加したスタイルになります。  
* `.toast`  
これはToastのデザインです。  
* `.toast:first-child`  
Toast要素は必ず1つ残り続けてしまうため、1番目の要素は非表示としています。  
* `.example-enter`  
Toastの表示し始めの透明度を設定しています。  
* `.example-enter.example-enter-active`  
Toast表示アニメーションになります。  
* `.example-leave`  
Toastの消え初めの透明度を設定しています。  
* `.example-leave-active`  
Toastの非表示アニメーションになります。  

####Counter ComponentのボタンからToast表示までの流れ
`Toast Component`の作成が完了したので、これを`Counter Component`から呼び出すように実装していきましょう。  

```javascript
// ./components/Counter.js
import React, { Component, PropTypes } from 'react';
import Toast from './Toast/Toast';

class Counter extends Component {
  static propTypes = {
    value: PropTypes.number,
    num: PropTypes.number.isRequired, // 追加 (1)
    onIncrement: PropTypes.func.isRequired,
    onDecrement: PropTypes.func.isRequired,
    onShowToast: PropTypes.func.isRequired  // 追加 (1)
  }

  // 省略...

  // 追加 (2)
  showToast = () => {
    this.props.onShowToast();
  }
  // 追加 (3)
  renderToast(num) {
    return (
      <Toast num={num} />
    );
  }
  render() {
    // 追加 (4)
    const { value, onIncrement, onDecrement, onShowToast, num } = this.props
    return (
      <p>
        Clicked: {value} times
        {' '}
        <button onClick={onIncrement}>
          +
        </button>
        {' '}
        <button onClick={onDecrement}>
          -
        </button>
        {' '}
        <button onClick={this.incrementIfOdd}>
          Increment if odd
        </button>
        {' '}
        <button onClick={this.incrementAsync}>
          Increment async
        </button>
        {' '}
        {/* 追加 (5) */}
        <button onClick={this.showToast}>
          Show Toast
        </button>
        {this.renderToast(num)}
      </p>
    )
  }
}

export default Counter
```

1つ1つ説明します。  

追加 (1):  
`num`は`Reducer`で変更された新しい`state`の`num`を受け付けるために`propTypes`に追加しました。  
`onShowToast`はボタンクリック時に`Toast`表示アクションの実行を受け付けるために`propTypes`に追加しました。  

追加 (2):  
ボタンアクションで実行するための追加です。  

追加 (3):  
`Toast Component`の描画メソッドです。  
1つ目の要素は非表示にCSSで設定しているので、初めから描画してしまって問題ありません。  

追加 (4):  
新たに`propTypes`として追加した`num`と`onShowToast`を利用するために書いています。  

追加 (5):  
ボタンクリック処理とToast描画処理を書いています。  
Toastので位置はCSSで右上固定にしてあるので、DOM位置がどこになろうと問題ありません。  

####Store周りの修正
最後に`Store`周りの修正です。  

```javascript
// ./index.js
import React from 'react'
import ReactDOM from 'react-dom'
import { createStore, bindActionCreators } from 'redux'
import { Provider, connect } from 'react-redux'
import Counter from './components/Counter'
import { increment, decrement, showToast } from './actions/index' // 追加 (1)
import counter from './reducers'

const store = createStore(counter)
const rootEl = document.getElementById('root')

function mapStateToPropsContainer (state) {
  return {
    value: state.value,
    num: state.num    // 追加 (2)
  }
}

function mapDispatchToPropsContainer (dispatch) {
  return {
    onIncrement: () => dispatch(increment()),
    onDecrement: () => dispatch(decrement()),
    onShowToast: () => dispatch(showToast())    // 追加 (3)
  }
}

let AppContainer = connect(
  mapStateToPropsContainer,
  mapDispatchToPropsContainer
)(Counter)

const render = () => ReactDOM.render(
  <Provider store={store}>
    <AppContainer />
  </Provider>,
  rootEl
)
render()
```

ここでは`mapStateToPropsContainer`と`mapDispatchToPropsContainer`を実装しているので、`Reducer`で新たに作成された`state`を受け取ります。  
それを適切に`Counter Component`に渡す必要があります。  

追加 (1):  
`Toast`表示アクションを`dispatch`に流し込む必要があるので、`showToast`を`action.js`から`import`しています。  

追加 (2):  
新たに生成された`state`の値を利用するために`mapStateToPropsContainer`に追記しています。  

追加 (3):  
`dispatch`に`Toast`表示アクションを流し込む設定を追記しています。  

###まとめ
さて如何でしたでしょうか？  
上記実装をすることで連打によるToast表示ができるようになったはずです。  
`ReactCSSTransitionGroup`周りを新たに学ぶ必要があったものの、基本的な`Redux`のデータフローを理解していればさほど難しくないことがわかったのではないでしょうか。  
筆者は結局のところ、`Redux`のデータフロー周りで躓くことが多く、不必要に難しく感じてしまっていました。  
アニメーションはWebサイト作りにおいて、なくてはならないものなので、これを機に`React`でのアニメーション作りの基礎を身に着けたいと思います。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
