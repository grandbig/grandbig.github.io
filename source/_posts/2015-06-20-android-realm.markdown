---
layout: post
title: "Android StudioでRealmを使ってみよう！"
date: 2015-06-20 23:56
comments: true
categories: android java
---

####Realmを読み込もう！
AndroidでもRealmを導入してみたいと思います。  
今後、SQLite同様、非常に重宝されるライブラリであることは間違いないですし。  

#####Realmの導入方法
ここに関しては、[公式サイト](https://realm.io/jp/docs/java/latest/)を見れば、特に困ることはないと思いますが、筆者と同様にAndroidアプリ開発の初心者は躓くかもしれないので、詳しく書いてみようと思います。  

まず、 **Realm Android** を検索すると、『 **Realm Java(0.☓☓.☓)** 』のような検索結果が出てくると思います。  
ここで注意して頂きたいのが、そのページが最新版のページになっているか否かということです。  
旧バージョンの場合、URLが **https://realm.io/jp/docs/java/0.☓☓.☓/** のようにバージョンが含まれています。  
最新版の場合は、URLが **https://realm.io/jp/docs/java/latest/** のように **latest** というワードが含まれています。  

現在(2015/06/20)は **0.80.3** が最新版となっています。  

では、導入していきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

１. GradleScripts > build.gradle(Module: app) のdependenciesを修正
`compile 'io.realm:realm-android:0.80.3'`を`dependencies { ... }`に追加します。  
２. メニューのFile > Project Structure... > Modulesのapp > Build Types > release > Minify Enabledをtrueに変更
![Minify Enabledを設定](/images/android-realm1.png)  
これにより、GradleScripts > build.gradle(Module: app)に

```java
android {
	...
	buildTypes {
		release {
			minifyEnabled true
			proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
		}
	}
	...
}
```

という記述が追加されます。  
３. GradleScripts > proguard-rules.proに記述を追加

```java
-keepnames public class * extends io.realm.RealmObject
-keep class io.realm.** { *; }
-dontwarn javax.**
-dontwarn io.realm.**
```

これでRealmを利用する準備が整いました。  
実際にコードを書いていきましょう。  

#####保存オブジェクトの生成
Realmでは保存する対象を`RLMObject`として作成します。  
SQLiteでいうところのテーブルを生成しているイメージでしょうか。  

今回は練習として`Engineer.java`内で`Engineer`クラスを作ってみました。  

```java
// Engineer.java
public class Engineer extends RealmObject {
	
	@PrimaryKey
	private int id;
	private int level;
	private String name;
	private String businessTitle;
	private RealmList<Skill> skills;
	private Date createDateTime;
	private Date updateDateTime;

	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}

	public int getLevel() {
		return level;
	}
	public void setLevel(int level) {
		this.level = level;
	}

	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}

	public String getBusinessTitle() {
		return businessTitle;
	}
	public void setBusinessTitle(String businessTitle) {
		this.businessTitle = businessTitle;
	}

	public RealmList<Skill> getSkills() {
		return skills;
	}
	public void setSkills(RealmList<Skill> skills) {
		this.skills = skills;
	}

	public Date getCreateDateTime() {
		return createDateTime;
	}
	public void setCreateDateTime(Date createDateTime) {
		this.createDateTime = createDateTime;
	}

	public Date getUpdateDateTime() {
		return updateDateTime;
	}
	public void setUpdateDateTime(Date updateDateTime) {
		this.updateDateTime = updateDateTime;
	}
}
```

```java
// Skill.java
public class Skill extends RealmObject {

	@PrimaryKey
	private int id;
	private String skill;

	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}

	public String getSkill() {
		return skill;
	}
	public void setSkill(String skill) {
		this.skill = skill;
	}
}
```

ポイントとしては、`skills`のように配列の形で値を保持したい場合は`RealmList`を利用するということです。  

#####オブジェクトのセレクト/インサート

続いて先ほど生成したオブジェクトをセレクト/インサートしてみましょう。  

```java
// MainActivity.java
public class MainActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		Realm realm = Realm.getInstance(this);
		RealmQuery<Engineer> query = realm.where(Engineer.class);
		query.equalTo("name", "ABC123");
		RealmResults<Engineer> result = query.findAll();

		if(result.size() > 0) {
			// 検索結果が見つかった場合
			Engineer engineer = result.first();
			int level = engineer.getLevel();
			String name = engineer.getName();
			String businessTitle = engineer.getBusinessTitle();
			RealmList<Skill> skills = engineer.getSkills();
			Date createDateTime = engineer.getCreateDateTime();

			Log.d("MainActivity", "name: " + name + ", businessTitle: " + businessTitle);
		} else {
			// 検索結果が見つからなかった場合
			realm.beginTransaction();
			Engineer engineer = realm.createObject(Engineer.class);
			engineer.setLevel(1);
			engineer.setName("ABC123");
			engineer.setBusinessTitle("新米エンジニア");

			Skill skill = realm.createObject(Skill.class);
			skill.setSkill("Java");
			engineer.getSkills().add(skill);
			engineer.setCreateDateTime(new Date());

			realm.commitTransaction();
		}
	}
	....
}
```

ここでは`name = "ABC123"`のデータを検索して、見つからない場合は新規データをインサートし、見つかった場合はデータをログに出力しています。  
書込みには`beginTransaction()`と`commitTransaction()`を利用しています。  

#####データの確認
Realm Browserを使いたいところですが、現在調査中です...。  
デバイス内のどこにファイルがあるのかについてはわかったのですが、うまくファイルをPCに取り込めません...。  
継続調査をし、ブログにアップしますね。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

