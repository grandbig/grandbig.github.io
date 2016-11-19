---
layout: post
title: "FMDBを使って簡単にiPhoneのローカルストレージを活用しよう！"
date: 2013-11-30 19:22
comments: true
categories: ios SQLite
---

###iPhoneのローカルストレージを開発で活用しよう！
本日はiPhoneのローカルストレージ活用方法について簡単に紹介したいと思います。  
iPhoneアプリを開発するにあたって、データ保管のためにDBサーバを活用したくなることがあると思います。しかし、個人で開発するにはレンタルサーバを借りるだけでも一苦労...。毎月お金もかかる...。そもそも保存したいデータ量も大きくないし...。なんてことがあるでしょう。そんなときは迷わず、iPhoneのローカルストレージを使いましょう！！  

<!--more-->

####FMDBを使おう！！
iPhoneローカルストレージを使うなら、非常に便利な**FMDB**をぜひダウンロードしましょう。  
FMDBはMITライセンスということもあり、商用でも簡単に導入できます。  

では早速、使い方を説明しましょう。  
1: GitHubより[FMDB](https://github.com/ccgus/fmdb)をダウンロードします。  
2: fmdb-master/src/ 配下のファイルをコピーしてFMDBを利用したいXcodeのプロジェクトフォルダ内に置く  
※ただし、fmdb.mファイルは不要  
![fmdb.m以外を利用する](/images/fmdatabase1.png)  
![fmdb.m以外をXcodeプロジェクトに追加](/images/fmdatabase2.png)  
3: libsqlite3.0.dylibフレームワークをXcodeプロジェクトに追加  
※これがないとエラーが出るので注意  
![libsqlite3.0.dylibフレームワークの追加](/images/fmdatabase3.png)  
4: FMDBを利用するファイルで**FMDatabase.h**をimportする  
![FMDatabase.hをimportする](/images/fmdatabase4.png)  

これでFMDBを利用することができます。  

####FMDBを使った具体的な書き方
次にFMDBを使った具体的な書き方について説明したいと思います。  

**DB接続の書き方**  
DB接続方法は下記だけです。
```
NSArray *paths = NSSearchPathForDirectoriesInDomains( NSDocumentDirectory, NSUserDomainMask, YES );
NSString *dir = [paths objectAtIndex:0];
FMDatabase *db = [FMDatabase databaseWithPath:[dir stringByAppendingPathComponent:@"test.db"]];
```
**DBの開閉**
```
//DBを開く場合
[db open];
//DBを閉じる場合
[db close];
```
上記がDBを扱う上での基本になります。  

**tableの作成**  
ここでは上記の基本を含めた一連の流れを書きます。  
下記ではTEXT型のuid, createtimeを持ったusersという名前のtableを作成しています。  
```
//DBの呼び出し
NSArray *paths = NSSearchPathForDirectoriesInDomains( NSDocumentDirectory, NSUserDomainMask, YES );
NSString *dir = [paths objectAtIndex:0];
FMDatabase *db = [FMDatabase databaseWithPath:[dir stringByAppendingPathComponent:@"test.db"]];

//tableの作成
NSString *sql = @"CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY,uid TEXT,createtime TEXT);";
[db open];
//クエリの実行
[db executeUpdate:sql];
[db close];
```

以下、説明することは上記で作成したtableを利用しています。  

**select文の実行**  
下記ではuidを取得する例を書きます。  
```
//DBの呼び出し
NSArray *paths = NSSearchPathForDirectoriesInDomains( NSDocumentDirectory, NSUserDomainMask, YES );
NSString *dir = [paths objectAtIndex:0];
NSString *dir = [paths objectAtIndex:0];
FMDatabase *db = [FMDatabase databaseWithPath:[dir stringByAppendingPathComponent:@"test.db"]];

//select文の作成
NSString *select = [[NSString alloc] initWithFormat:@"SELECT * from users"];
[db open];
FMResultSet *rs = [db executeQuery:select];
NSString *uid;
while([rs next]) {
	uid = [rs stringForColumnIndex:1];
}
[db close];
```

**insert文の実行**  
```
//DBの呼び出し
NSArray *paths = NSSearchPathForDirectoriesInDomains( NSDocumentDirectory, NSUserDomainMask, YES );
NSString *dir = [paths objectAtIndex:0];
NSString *dir = [paths objectAtIndex:0];
FMDatabase *db = [FMDatabase databaseWithPath:[dir stringByAppendingPathComponent:@"test.db"]];

//insert文の作成
NSString *createtime = @"20131130011728462";
NSString *uid = @"user01";
NSString *insert = [[NSString alloc] initWithFormat:@"INSERT INTO users(uid, createtime) VALUES('%@', '%@')", uid, createtime];
[db open];
[db executeUpdate:insert];
[db close];
```

**update文の実行**  
```
//DBの呼び出し
NSArray *paths = NSSearchPathForDirectoriesInDomains( NSDocumentDirectory, NSUserDomainMask, YES );
NSString *dir = [paths objectAtIndex:0];
NSString *dir = [paths objectAtIndex:0];
FMDatabase *db = [FMDatabase databaseWithPath:[dir stringByAppendingPathComponent:@"test.db"]];

//update文の作成
NSString *createtime = @"20131201054830467";
NSString *uid = @"user02";
//DBにレコードが１つ追加されていて、そのid=1のレコードを更新しています
NSString *update = [[NSString alloc] initWithFormat:@"UPDATE users SET uid='%@', createtime='%@' where id=1", uid, createtime];
[db open];
[db executeUpdate:update];
[db close];
```

**delete文の実行** 
```
//DBの呼び出し
NSArray *paths = NSSearchPathForDirectoriesInDomains( NSDocumentDirectory, NSUserDomainMask, YES );
NSString *dir = [paths objectAtIndex:0];
NSString *dir = [paths objectAtIndex:0];
FMDatabase *db = [FMDatabase databaseWithPath:[dir stringByAppendingPathComponent:@"test.db"]];

//delete文の作成
NSString *delete = [[NSString alloc] initWithFormat:@"DELETE from users where id=1"];
[db open];
[db executeUpdate:delete, [NSNumber numberWithInteger:n]];
[db close];
```

**count文の実行**  
```
//DBの呼び出し
NSArray *paths = NSSearchPathForDirectoriesInDomains( NSDocumentDirectory, NSUserDomainMask, YES );
NSString *dir = [paths objectAtIndex:0];
NSString *dir = [paths objectAtIndex:0];
FMDatabase *db = [FMDatabase databaseWithPath:[dir stringByAppendingPathComponent:@"test.db"]];

//count文の作成
NSString *uid = @"user02";
NSString *cntquery = [[NSString alloc] initWithFormat:@"select count(*) as count from users where uid=%d", uid];
[db open];
FMResultSet *cntrs = [db executeQuery:cntquery];
if([cntrs next]) {
	NSString *count = [cntrs intForColumn:@"count"];
}
[db close];
```

####まとめ
たったこれだけのことを知っておくだけでiPhoneのローカルストレージを使いこなすことができます。ぜひ使ってみてください！(筆者もめちゃくちゃ使っています。)  

参考:  
[iOS で SQLite – FMDB の使い方](http://akabeko.me/blog/2011/11/ios-sqlite-fmdb/)  
[Objective-Cと戦うブログ](http://obc-fight.blogspot.jp/2012/09/fmdb-basic-how-to-use.html)  
