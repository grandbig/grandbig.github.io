---
layout: post
title: "FMDBを使ったDB接続を関数化しよう！！"
date: 2014-04-11 22:17
comments: true
categories: ios SQLite
---

###FMDBを使ったDB接続を自作関数化
さて、ブログを更新します。以前、[FMDBを使って簡単にiPhoneのローカルストーレジを活用しよう！](http://grandbig.github.io/blog/2013/11/30/fmdatabase/)でObjective-CでSQLiteを扱うためのFMDBライブラリについて説明させて頂きました。  
そのときは全てをベタ書きしていましたが、毎回書くのは面倒ですよね...ってことで自作しちゃいましょう！！

<!--more-->

まずはおさらいです。DB接続は下記です。  
```objective-c
NSArray *paths = NSSearchPathForDirectoriesInDomains( NSDocumentDirectory, NSUserDomainMask, YES );
NSString *dir = [paths objectAtIndex:0];
FMDatabase *db = [FMDatabase databaseWithPath:[dir stringByAppendingPathComponent:@"test.db"]];
```
これをtableの作成, select, insert, update, delete, countのたびに書くのは面倒なので...

```objective-c
- (id)dbConnect:(NSString *)dbName
{
	NSArray *paths = NSSearchPathForDirectoriesInDomains( NSDocumentDirectory, NSUserDomainMask, YES );
	NSString *dir = [paths objectAtIndex:0];
	FMDatabase *db = [FMDatabase databaseWithPath:[dir stringByAppendingPathComponent:dbName]];

	return db;
}
```
としちゃいましょう。  
そうすれば、同じmファイル内であればtableの作成, select, insert, update, delete, countは以下のように書けます。  

####tableの作成
```objective-c
- (void)createTable:(NSString *)tableName
{
	FMDatabase *db = [self dbConnect:@"test.db"];
	
	// tableの作成
	NSString *sql = [[NSString alloc] initWithFormat:@"create table if not exists %@ (id INTEGER PRIMARY KEY, uid TEXT, createtime TEXT)", tableName];
	
	[db open];
	[db executeUpdate:sql];
	[db close];
}
```

####select文の実行
```objective-c
- (void)selectData:(NSString *)tableName
{
	FMDatabase *db = [self dbConnect:@"test.db"];
	
	// select文
	NSString *sql = [[NSString alloc] initWithFormat:@"select * from %@", tableName];

	[db open];

	FMResultSet *rs = [db executeQuery:sql];
	while([rs next]) {
		NSInteger idNum = [[rs stringForColumnIndex:0] intValue];
		NSString *uid = [rs stringForColumnIndex:1];
		NSString *createtime = [rs stringForColumnIndex:2];
		NSLog(@"id: %ld, uid: %@, createtime: %@", (long)idNum, uid, createtime);
	}

	[db close];
}
```
####insert文の実行
```objective-c
- (void)insertData:(NSString *)tableName
			userID:(NSString *)uid
		createtime:(NSString *)createtime
{
	FMDatabase *db = [self dbConnect:@"test.db"];
	
	// insert文
	NSString *sql = [[NSString alloc] initWithFormat:@"insert into %@ (uid, createtime) values('%@', '%@')", tableName, uid, createtime];

	[db open];
	[db executeUpdate:sql];
	[db close];
}
```

####update文の実行
```objective-c
- (void)updateData:(NSString *)tableName
			userID:(NSString *)uid
		createtime:(NSString *)createtime
{
	FMDatabase *db = [self dbConnect:@"test.db"];
	
	// update文
	NSString *sql = [[NSString alloc] initWithFormat:@"update %@ set uid='%@', createtime='%@' where id=1", tableName, uid, createtime];

	[db open];
	[db executeUpdate:sql];
	[db close];
}
```
####delete文の実行
```objective-c
- (void)deleteData:(NSString *)tableName
{
	FMDatabase *db = [self dbConnect:@"test.db"];
	
	// delete文
	NSString *sql = [[NSString alloc] initWithFormat:@"delete from %@", tableName];

	[db open];
	[db executeUpdate:sql];
	[db close];
}
```
####count文の実行
```objective-c
- (NSInteger)countData:(NSString *)tableName
{
	FMDatabase *db = [self dbConnect:@"test.db"];

	// count文
	NSString *sql = [[NSString alloc] initWithFormat:@"select count(*) as count from %@", tableName];
	
	[db open];
	FMResultSet *rs = [db executeQuery:sql];
	NSInteger cnt = 0;
	if([rs next]) {
		cnt = [rs intForColumn:@"count"];
	}
	[db close];

	return cnt;
}
```

一度、関数を作ってしまえば、使いやすさがぐんぐん増します！！  
自身のiOSアプリに最適なSQLiteを扱うメソッドを作ってみてはどうでしょうか？  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
