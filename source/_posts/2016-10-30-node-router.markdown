---
layout: post
title: "Node.jsでrouterモジュールでルーティング！"
date: 2016-10-30 00:00
comments: true
categories: node javascript BE
---

###Expressなしで簡単にルーティングしよう！
さて、久しぶりにNode.jsについて書きます。  
筆者が本格的にNode.jsを利用していたのは3〜4年前だったため、Expressをよく利用していました。  
その後、Expressを利用するほどのリッチな機能を必要としない、簡易的なデモ用のサーバサイドの仕組みを作るのにバリバリ自作ルーティングをしていました。  
しかし、ここにきてExpressを利用するでもなく、かと言って自作でルーティング処理を書くのも若干面倒だと感じるとき果たしてどうすれば良いのかふと気になりました。  
「きっと今の世の中なら何らかのモジュールが出ているはず！」と思った筆者は早速探してみることに...  

そこで見つけたのが[router](https://github.com/pillarjs/router)です。  
このモジュールを利用すれば、次のように簡単にルーティングを実装することができます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
<!-- more -->

```javascript
// server.js
var http = require("http");
var Router = require("router");
var finalhandler = require('finalhandler');
var bodyParser   = require('body-parser');

function start(route) {
	var opts = { mergeParams: true }
	var router = Router(opts);
	var server = http.createServer(function onRequest(req, res) {
		router(req, res, finalhandler(req, res));
	});

	// GETリクエストのハンドリング
	var user = Router(opts);
	router.use('/users/:path', user);

	user.get('/', function (req, res) {
		res.statusCode = 200
		res.setHeader('Content-Type', 'text/plain; charset=utf-8')
		res.end(req.params.path + '\n')
	});

	// POSTリクエストのハンドリング
	var users = Router();
	router.use('/users', users);
	users.use(bodyParser.json());

	users.post('/', function (req, res) {
		if (req.body.value) {
			res.statusCode = 200;
			res.setHeader('Content-Type', 'text/plain; charset=utf-8');
			res.end(req.body.value + '\n');
		} else {
			res.statusCode = 400;
			res.setHeader('Content-Type', 'text/plain; charset=utf-8');
			res.end('Invalid API Syntax\n');
		}
	});

	server.listen(8888);
	console.log("server has started.");
}

exports.start = start;
```

因みに、 `server.js` はモジュールとして切り出しています。  
実際には `app.js` で呼び出すようにしています。  

```javascript
// app.js
var server = require("./server");
server.start();
```

ただ、今後たくさんのリクエストを捌くことを考えると、もう少しリクエスト内容ごとにファイルを分けた方が良いですよね...  
と言うことで少々修正します。  

```javascript
// server.js
var http = require("http");
var Router = require("router");
var finalhandler = require('finalhandler');
var bodyParser   = require('body-parser');
var users = require("./users");

/**
 *	サーバ起動処理
 */
function start() {
	var opts = { mergeParams: true };
	var router = Router(opts);
	var server = http.createServer(function onRequest(req, res) {
		router(req, res, finalhandler(req, res));
	});

	router.use(bodyParser.json());
	router.use('/users/:path', users);	// GETリクエスト
	router.use('/users', users);		// POST, PUT, DELETEリクエスト

	server.listen(8888);
	console.log("server has started.");
}

exports.start = start;

```

上記のように`server.js`はサーバ起動だけに絞りました。  
そしてリクエストを受け付けたあとの処理は下記のように`users.js`に書きます。  

```javascript
// users.js
var Router = require("router");
var opts = { mergeParams: true };
var router = Router(opts);

router.get("/", function(req, res) {
	res.statusCode = 200;
	res.setHeader("Content-Type", "text/plain; charset=utf-8");
	res.end(req.params.path + "\n");
});

router.post("/", function(req, res) {
	if (req.body.value) {
		res.statusCode = 200;
		res.setHeader('Content-Type', 'text/plain; charset=utf-8');
		res.end(req.body.value + '\n');
	} else {
		res.statusCode = 400;
		res.setHeader('Content-Type', 'text/plain; charset=utf-8');
		res.end('Invalid API Syntax\n');
	}
});

module.exports = router;

```

このようにまとめることで可読性高くなるので、ルーティングの意味も出てくるというものですね。  
因みに、GETとPOSTリクエストを送れば下記のような結果が得られます。  

```
// GETリクエスト
curl http://localhost:8888/users/20161029
// 結果
20161029

// POSTリクエスト
curl http://localhost:8888/users -X POST -H "Content-Type: application/json" -d '{"value":"Sample"}'
// 結果
Sample
```

###Node.jsでデバッグ
筆者はこれまで `node-inspector` を利用していたのですが、何とv6.3.0からデバッグ機能が標準装備されているらしいですね！  
早速ですが使ってみました。  

```
// Node.jsでデバッグ起動
node --inspect app.js
Debugger listening on port 9229.
Warning: This is an experimental feature and could change at any time.
To start debugging, open the following URL in Chrome:
    chrome-devtools://devtools/remote/serve_file/@60cd6e859b9f557d2312f5bf532f6aec5f284980/inspector.html?experiments=true&v8only=true&ws=localhost:9229/f1478fd8-33f2-4bca-8ab4-4ac9be3515cb
server has started.
Debugger attached.
```

出力される `chrome-devtools://devtools/remote/serve_file/@....` の部分をChromeのアドレスバーに貼りましょう！  
`node-inspcetor` さながらのデバッグができるはずです。  

因みに、起動時のオプションとして `--debug-brk` をつけると必ず1行目でデバッグが停止します。  
一度停めたい場合はオプションを使いましょう。  

###まとめ
今回はNode.jsに触る機会があったため、どうすれば簡単にルーティングが実装できるのか調べてみました。  
Expressを使っても良かったのですが、極力不要なモジュールを取り込みたくない気持ちがあったので割りと最低限にできて良かったなと思いました。  
Node.jsは少しずつリハビリしながら思い出していくことにしようかな。  
と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
