---
layout: post
title: "GitHubとSlackを自作で連携させてみよう！"
date: 2019-05-26 11:47
comments: true
categories: github slack node
---

### はじめに
今回は `GitHub` と `Slack` の連携を自作でやってみる話の備忘録です。  

と言っても、手法は本当に簡易で、  

* `GitHub` 上で `Webhook` を設定する  
* `GitHub` からの `POSTリクエスト` を受け付けるサーバを用意する  
* 受け付けた `POSTリクエスト` から各種値を抽出して、Slackに通知する   

を対応することで実現することが可能です。  
それでは早速見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

### GitHub上でWebhookを設定する
`GitHub` 上で `Slack` と連携したい `Repository` を開いて、`Settings` ページにアクセスします。  
`Settings` ページの左メニューから `Webhooks` を選択して、`Add webhook` を選択します。  

![Add webhookを選択](/images/github2slack_1.png)  

続いて、`Add webhook` ページを編集します。  

![Add webhookの編集](/images/github2slack_2.png)  

① `Payload URL` にPOSTリクエストの送り先を設定します  
② `Content type` は `application/json` を設定します  
③ `Which events would you like to trigger this webhook?` に設定したいものを設定します  
　 ※筆者の場合は、個別に細かく設定可能な `Let me select individual events.` を選択しています。  

### GitHubからのPOSTリクエストを受け付けるサーバを用意する
ここは開発者の好きなインフラを用意すれば良いと思います。  
AWSでもGCPでも良いかと思います。  

また、APIサーバの作り込みに関しても、好きな言語で、自由にフレームワーク等を使って実装すれば良いと思います。  
ポイントは、時間をかけずに作ることです。  
(あくまでもチーム開発などを円滑に進めるための取り組みであって、この作り込みが目的ではないためです。)  

因みに、筆者は、`Node.js`と`Express`を利用してAPIサーバを作ることにしました。  

### 受け付けたPOSTリクエストから各種値を抽出して、Slackに通知する
では、少しAPIサーバの実装について紹介します。  

#### package.json
まず、`package.json`は下記のようにしました。  

```javascript
{
  "name": "github2slack",
  "version": "1.0.0",
  "description": "GitHub Action Notification",
  "main": "server.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "dependencies": {
    "@slack/web-api": "^5.0.1",
    "express": "^4.17.0"
  }
}
```

先程申し上げた通り `Express` を用いているのと、`Slack Web API`を簡単に利用できる `@slack/web-api`も導入しています。  

#### server.js
続いてメインとなる`server.js`です。  

`POSTリクエスト`を受け付けるところまでですが、下記のように実装しています。  
※ただ、`Express`を利用している、よくある書き方かなと思います。  

```javascript
var https = require('https');
var express = require('express');
var bodyParser = require('body-parser')
const { WebClient, LogLevel, retryPolicies } = require('@slack/web-api');
const slackChannel = 'XXXXXXXX';  // 通知先のSlackチャンネルのID
const serverPort = 3000;

const app = express();
app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json());

/// POSTリクエストの受付
app.post('/', (req, res) => {
  ...
});

/// サーバを起動
app.listen(serverPort, () => console.log('Listening on port ' + serverPort));
```

肝心の`POSTリクエスト`で受け付けた後の処理ですが、  

```javascript
app.post('/', (req, res) => {
  // GitHubからのリクエストを分析
  let message = makeMessageFromGitHubRequest(req) ---> 後ほど説明します
  if (!message) {
    // 送るべきメッセージがないため、200で返して終了する
    res.status(200).send('no message');
    return;
  }

  // Slackにメッセージを投稿
  const slack = new WebClient('', {
    slackApiUrl: 'http://xxxxxxxxxxxxxxxxxxxxxx',
    retryConfig: retryPolicies.tenRetriesInAboutThirtyMinutes,
    logLevel: LogLevel.DEBUG
  })

  const main = async () => {
    try {
      const result = await slack.chat.postMessage({
        channel: slackChannel,
        text: message
      })

      res.status(200).send('message: ' + message + '\n');
    } catch (err) {
      res.status(500).send(err.message + '\n');
    }
  }

  main()
});
```

のようになっています。  
先程導入した `@slack/web-api` を利用すれば、いとも簡単に`Slack`への通知が可能となっています。   

そして`Slack`にどんな通知を送るかについてですが、`GitHub`上で発生したイベント毎にメッセージが異なると考えられます。  
よって、`GitHub`から`POSTリクエスト`された値を分析して、最適なメッセージを送る処理を導入してみましょう。  

今回、筆者は下記のイベントをハンドリングすることにしました。  

* `PullRequest`に`Reviewer`をセットした時に、レビュー依頼を流す  
* `PullRequest`のレビューをして、ステータスを変更した時に、レビュー依頼者に知らせる  
* `PullRequest`の`diff`に沿ってコメントを書いたことを流す  
* `PullRequest`の`Conversation`にコメントを書いたことを流す  

それらをハンドリングするメソッドをまずは用意します。  

```javascript
/**
 * GitHubからのリクエストの中身を精査して、メッセージを作成する
 * @param {object}  req   POSTリクエスト
 * @return {string}       メッセージ
 */
function makeMessageFromGitHubRequest(req) {
  const gitHubEvent = req.headers['x-github-event'];
  const gitHubBody = req.body;

  if (gitHubEvent === 'pull_request') {
    return makePullRequestMessage(gitHubBody.action, gitHubBody);
  } else if (gitHubEvent === 'pull_request_review') {
    return makeChangeReviewStateMessage(gitHubBody.action, gitHubBody);
  } else if (gitHubEvent === 'pull_request_review_comment') {
    return makePullRequestReviewCommentMessage(gitHubBody.action, gitHubBody.comment);
  } else if (gitHubEvent === 'issue_comment') {
    return makeIssueCommentMessage(gitHubBody.action, gitHubBody.comment);
  } else {
    return null;
  }
}
```

それぞれイベントによって作成するメッセージが異なるため、1つずつメソッドを切り出しました。  
『`PullRequest`に`Reviewer`をセットした時に、レビュー依頼を流す』場合の処理は下記です。  

```javascript
/**
 * PullRequest関連のメッセージを作成する
 * @param {string}  action      アクション種別
 * @param {object}  body        GitHubのBody情報
 * @return {string}             メッセージ
 */
function makePullRequestMessage(action, body) {
  let mention;
  let from;
  let content;
  let title = body.pull_request.title;

  let message;
  let linkUrl = body.pull_request.html_url;

  switch(action) {
    case 'created':
      mention = makeMentionTextAtPullRequestCreated(body.pull_request);
      from = '「' + body.pull_request.user.login + '」さんから';
      content = 'PullRequestのレビュー依頼が届きました\n\n';
      message = mention + from + content + title;
      break;
    case 'review_requested':
      mention = makeMentionText(body.requested_reviewer);
      from = '「' + body.pull_request.user.login + '」さんから';
      content = 'PullRequestのレビュー依頼が届きました\n\n';
      message = mention + from + content + title;
      break;
    default:
      break;
  }

  if (!message) {
    return null;
  }
  return message + '\n' + linkUrl;
}

/**
 * PullRequest作成時にレビューを要求された場合にメンションテキストを作成する処理
 * @param {object} pullRequest  PullRequest情報
 * @return {string}             メンションテキスト
 */
function makeMentionTextAtPullRequestCreated(pullRequest) {
  let result = '';
  const requested_reviewers = pullRequest.requested_reviewers;
  requested_reviewers.forEach(reviewer => {
    // Slack APIでは、<@username>の形でメッセージを作らないとメンションできない
    const mention = "<@" + reviewer.login + "> ";
    result = result + mention;
  });
  result = result + '\n';
  return result;
}

/**
 * PullRequestレビュワー追加時にメンションテキストを作成する処理
 * @param {object} user  ユーザ情報
 * @return {string}      メンションテキスト
 */
function makeMentionText(user) {
  const mention = "<@" + user.login + "> ";
  return mention + '\n';
}
```

『`PullRequest`のレビューをして、ステータスを変更した時に、レビュー依頼者に知らせる』場合は下記です。  

```javascript
/**
 * PullRequestのReview関連のメッセージを作成する
 * @param {string}  action  アクション種別
 * @param {object}  body    GitHubのBody情報
 * @return {string}         メッセージ
 */
function makeChangeReviewStateMessage(action, body) {
  let mention;
  let from;
  let content;
  let title = convertCommentWithMention(body.review.body);

  let message;
  let linkUrl = body.review.html_url;

  switch(action) {
    case 'submitted':
      mention = makeMentionText(body.pull_request.user);
      from = '「' + body.review.user.login + '」さんから'
      content = 'レビュー結果： *' + body.review.state + '* が返ってきました。\n\n';
      message = mention + from + content + title;
      break;
    default:
      break;
  }

  if (!message) {
    return null;
  }
  return message + '\n' + linkUrl;
}

/**
 * コメント内にメンションがあった場合に、Slackが検知できるメンションテキストに置き換える処理
 * @param {object} comment  コメント
 * @return {string}         メンションテキスト
 */
function convertCommentWithMention(comment) {
  return comment.replace(/@[A-Za-z0-9-/]+/g, "<$&>");
}
```

『* `PullRequest`の`diff`に沿ってコメントを書いたことを流す』場合は下記です。  

```javascript
/**
 * PullRequestのReviewコメント関連のメッセージを作成する
 * @param {string}  action  アクション種別
 * @param {object}  comment コメント情報
 * @return {string}         メッセージ
 */
function makePullRequestReviewCommentMessage(action, comment) {
  let from;
  let content;
  let title = convertCommentWithMention(comment.body);

  let message;
  let linkUrl = comment.html_url;

  switch(action) {
    case 'created':
      // PullRequestのFileChanges内でコメントした場合
      from = '「' + comment.user.login + '」さんから'
      content = 'コメントが届きました。\n\n';
      message = from + content + title;
      break;
    default:
      break;
  }

  if (!message) {
    return null;
  }
  return message + '\n' + linkUrl;
}
```

『* `PullRequest`の`Conversation`にコメントを書いたことを流す』場合は下記です。  

```javascript
/**
 * PullRequestやIssueのConversation欄のコメント関連のメッセージを作成する
 * @param {string}  action  アクション種別
 * @param {object}  comment コメント情報
 * @return {string}         メッセージ
 */
function makeIssueCommentMessage(action, comment) {
  let from;
  let content;
  let title = convertCommentWithMention(comment.body);

  let message;
  let linkUrl = comment.html_url;

  switch (action) {
    case 'created':
      from = '「' + comment.user.login + '」さんから';
      content = 'コメントが届きました。\n\n';
      message = from + content + title;
      break;
    default:
      break;
  }

  if (!message) {
    return null;
  }
  return message + '\n' + linkUrl;
}
```

### まとめ
さて、割と手軽に実装できることがわかったところで本日はここまで。  
今後、もっと様々なイベントをキャッチして`Slack`に流す必要性が出てきたら、どんどん足していこうかな。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
