---
layout: post
title: "FMDBを使ってSQLiteに画像を保存しよう！"
date: 2015-05-07 00:37
comments: true
categories: ios SQLite
---

####FMDBを使ってSQLiteに画像を保存してみる
さて、当たり前なようでやったことがなかったのでやってみました。  
たぶんもっとキレイな方法があると思うのですが、なぜかできず...。できた方法を説明します。  

#####SQLiteにデータを保存
まず下記のメソッドを実装します。  

```objective-c
// DBに接続
- (id)dbConnect:(NSString *)dbName
{
	NSArray *paths = NSSearchPathForDirectoriesInDomains( NSDocumentDirectory, NSUserDomainMask, YES );
	NSString *dir = [paths objectAtIndex:0];
	FMDatabase *db = [FMDatabase databaseWithPath:[dir stringByAppendingPathComponent:dbName]];

	return db;
}

// テーブルの作成
- (void)createTable:(NSString *)dbName tableName:(NSString *)tableName
{
	FMDatabase *db = [self dbConnect:dbName];
	NSString *sql = [[NSString alloc] initWithFormat:@"create table if not exists %@ (id INTEGER PRIMARY KEY, title TEXT, imgData TEXT)", tableName];
	[db open];
	[db executeUpdate:sql];
	[db close];
}

```

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

さて、上記のメソッドは基本的なDB接続とテーブル作成をするためのものです。  
続いて画像のデータをinsertします。  

```objective-c
- (void)insertImgData:(NSString *)dbName tableName:(NSString *)tableName
{
	UIImage *img = [UIImage imageNamed:@"apple.png"];
	NSData *imgData = [NSData alloc] initWithData:UIImagePNGRepresentation(img)];
	NSString *imgString = [imgData base64EncodedStringWithOptions:NSDataBase64Encoding76CharacterLineLength];
	FMDatabase *db = [self dbConnect:dbName];
	NSString *insert = [[NSString alloc] initWithFormat:@"insert into %@ (title, imgData) values ('%@', '%@')", tableName, @"test", imgString];
	[db open];
	[db executeUpdate:insert];
	[db close];
}

```

今回はテストのために固定の画像を内部DBにinsertするようにしています。  
そして、画像データを取得するメソッドを用意します。  

```objective-c
- (UIImage)getImgData:(NSString *)dbName tableName:(NSString *)tableName
{
	NSData *imgData;
	FMDatabase *db = [self dbConnect:dbName];
	NSString *select = [[NSString alloc] initWithFormat:@"select * from %@ order by id desc limit 1", table_name];
	[db open];
	FMResultSet *rs = [db executeQuery:select];
	while([rs next]) {
		NSString *imgString = [rs stringForColumn:@"imgData"];
		imgData = [[NSData alloc] initWithBase64EncodedString:imgString options:NSDataBase64DecodingIgnoreUnknownCharacters];
	}
	UIImage *img = [UIImage alloc] initWithData:imgData];]
}
```

重要なのはエンコード/デコードです。  
いろいろなサイトを見ると、NSData型の値をSQLiteのBLOB型で保存すれば良いと書かれているのですが、デフォで画像用でないエンコードされているからなのか、想定した値が取れないんですよね〜。  
う〜ん。もう少し考えてみますが、一先ず、これでもできるってことでメモ書きとしておきます。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
