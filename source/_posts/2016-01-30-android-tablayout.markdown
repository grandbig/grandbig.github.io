---
layout: post
title: "Android Studio2.0: TabLayoutを使ってみよう！"
date: 2016-01-30 14:32
comments: true
categories: android java
---

####TabHostとTabLayoutについて
前回、AndroidでTabHostを使ったアプリの作り方について見ていきましたが、本日は **TabLayout** を使ったアプリの作り方を見ていきたいと思います。  
なぜ似たような名前のクラスがあるのかと思いますよね？  
実はTabHostはもう随分前から非推奨とされており、同様の機能を実装するために **TabLayout** が提供されているようなんです。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

もう少し、詳しく説明すると、  
『Google I/O 2014』にて[Material Design](https://www.google.com/design/spec/material-design/introduction.html)が発表されました。  
(これはGoogleが提唱するパソコンやスマートフォン向けのアプリまたはWebサイト等の新しいデザイン方式です。)  

そして、『[Google I/O 2015](https://events.google.com/io2015/#)』にてMaterial Designを実現するためのサポートライブラリ『 **Android Design Support Library** 』が発表されました。  
**TabLayout** はこの **Android Design Support Library** に含まれるコンポーネントの1つなのです。  

また、現在(2016/01/30時点)、公式でリリースされているAndroid Studioでは自身で`build.gradle`に`compile 'com.android.support:design:23.+'`と追記する必要がありますが、Preview版として提供されている **Android Studio 2.0** では、Blank Activityですらデフォルトで **Android Design Support Library** を利用するように組み込まれています。  

さて、今日はそんな **Android Design Support Library** のコンポーネントの1つ **TabLayout** について見ていきたいと思います。  

<!-- more -->

####TabLayoutを使ってみよう！
TabHostを書いた時と同じ書式で書いていきます。  

1． Android Studioで`Blank Activity`を選択して、プロジェクトを新規作成  
下記のようなプロジェクト構成ができあがります。  

```java
app
 ┣━━ manifests
 ┃    ┗━━ AndroidManifest.xml
 ┣━━ java
 ┃    ┣━━ com.example.takahiro.tabhostapplication
 ┃    ┃      ┗━━ MainActivity
 ┃    ┣━━ com.example.takahiro.tabhostapplication(androidTest)
 ┃    ┃      ┗━━ ApplicationTest
 ┃    ┗━━ com.example.takahiro.tabhostapplication(test)
 ┃           ┗━━ ExampleUnitTest
 ┗━━ res
      ┣━━ drawable
      ┣━━ layout
      ┃     ┣━━ activity_main.xml
      ┃     ┗━━ content_main.xml
      ┣━━ menu
      ┣━━ mipmap
      ┗━━ values
```


2．`content_main.xml`にTabLayoutを追加  
ページ遷移のためにViewPagerも追加しておきます。  

```java
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	....
	tools:showIn="@layout/activity_main">

	<android.support.design.widget.TabLayout
		android:id="@+id/tabs"
		android:layout_width="match_parent"
		android:layout_height="wrap_content" />

	<android.support.v4.view.ViewPager
		android:id="@+id/pager"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:layout_below="@+id/tabs" />
</RelativeLayout>
```

因みに、`activity_main.xml`は以下にしています。  

```java
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:app="http://schemas.android.com/apk/res-auto"
	xmlns:tools="http://schemas.android.com/tools"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:fitsSystemWindows="true"
	tools:context="com.example.takahiro.tablayoutsample2.MainActivity">

	<android.support.design.widget.AppBarLayout
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:theme="@style/AppTheme.AppBarOverlay">

		<android.support.v7.widget.Toolbar
			android:id="@+id/toolbar"
			android:layout_width="match_parent"
			android:layout_height="?attr/actionBarSize"
			android:background="?attr/colorPrimary"
			app:popupTheme="@style/AppTheme.PopupOverlay" />

	</android.support.design.widget.AppBarLayout>

	<include layout="@layout/content_main" />
</android.support.design.widget.CoordinatorLayout>
```

デフォルトで`FloatingActionButton`が作成されますが、こちらは今回不要なので削除しています。  

3．表示ページ用に`Fragment`を継承した`PageFragment`を作成  
メニューから`BlankFragment`を追加すると、ある程度デフォルトで処理を作成してくれます。  
まずは`PageFragment.java`ファイルは下記に一部修正しました。  

```java
package com.example.takahiro.tablayoutapplication;

import android.content.Context;
import android.net.Uri;
import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

public class PageFragment extends Fragment {

	private static final String ARG_PARAM = "page";
	private String mParam;
	private OnFragmentInteractionListener mListener;
	
	// コンストラクタ
	public PageFragment() {
	}

	public static PageFragment newInstance(int page) {
		PageFragment fragment = new PageFragment();
		Bundle args = new Bundle();
		args.putInt(ARG_PARAM, page);
		fragment.setArguments(args);
		return fragment;
	}
	
	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		if (getArguments() != null) {
			mParam = getArguments().getString(ARG_PARAM);
		}
	}

	@Override
	public View onCreateView(LayoutInflater inflater, ViewGroup container,
		Bundle savedInstanceState) {
		// Inflate the layout for this fragment
		int page = getArguments().getInt(ARG_PARAM, 0);
		View view = inflater.inflate(R.layout.fragment_page, container, false);
		(TextView)view.findViewById(R.id.textView)).setText("Page" + page);
		
		return view;
	}

	public void onButtonPressed(Uri uri) {
		if (mListener != null) {
			mListener.onFragmentInteraction(uri);
		}
	}

	@Override
	public void onAttach(Context context) {
		super.onAttach(context);
		if (context instanceof OnFragmentInteractionListener) {
			mListener = (OnFragmentInteractionListener) context;
		} else {
			throw new RuntimeException(context.toString()
				+ " must implement OnFragmentInteractionListener");
		}
	}

	@Override
	public void onDetach() {
		super.onDetach();
		mListener = null;
	}

	public interface OnFragmentInteractionListener {
		void onFragmentInteraction(Uri uri);
	}
}
```

続いて、XMLファイルは下記です。  

```java
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:tools="http://schemas.android.com/tools"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	tools:context="com.example.takahiro.tablayoutapplication.PageFragment">
	
	<TextView
		android:layout_width="wrap_content"
		android:layout_height="wrap_content"
		android:text="Page"
		android:id="@+id/textView"
		android:layout_centerVertical="true"
		android:layout_centerHorizontal="true"
		android:textSize="32sp" />
</RelativeLayout>
```

4．`MainActivity.java`にTabLayoutとViewPagerを組み込み  

```java
public class MainActivity extends AppCompatActivity implements ViewPager.OnPageChangeListener, 
		PageFragment.OnFragmentInteractionListener {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
		setSupportActionBar(toolbar);

		// xmlからTabLayoutの取得
		TabLayout tabLayout = (TabLayout) findViewById(R.id.tabs);
		// xmlからViewPagerを取得
		ViewPager viewPager = (ViewPager) findViewById(R.id.pager);
		// ページタイトル配列
		final String[] pageTitle = {"HOME", "EVENT", "SETTING"};

		// 表示Pageに必要な項目を設定
		FragmentPagerAdapter adapter = new FragmentPagerAdapter(getSupportFragmentManager()) {
			@Override
			public Fragment getItem(int position) {
				return PageFragment.newInstance(position + 1);
			}

			@Override
			public CharSequence getPageTitle(int position) {
				return pageTitle[position];
			}

			@Override
			public int getCount() {
				return pageTitle.length;
			}
		};

		// ViewPagerにページを設定
		viewPager.setAdapter(adapter);
		viewPager.addOnPageChangeListener(this);

		// ViewPagerをTabLayoutを設定
		tabLayout.setupWithViewPager(viewPager);
	}
	....

	@Override
	public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
	}

	@Override
	public void onPageSelected(int position) {
	}

	@Override
	public void onPageScrollStateChanged(int state) {
	}
	
	@Override
	public void onFragmentInteraction(Uri uri) {
	}
}
```

これでタブによる画面遷移が完成です。  

![TabLayoutによるアプリ画面](/images/android-tablayout.png)  

####スワイプを禁止した単純なTab遷移にするためには
筆者のTabの印象には、横スワイプによる画面切替は含まれていませんでした。  
なので、スワイプによる画面切替を禁止してみましょう。  
[HoldableViewPager](http://pastebin.com/J63dvdDB)をまんま使います。  

`ViewPager`を継承した`HoldableViewPager`を作成します。  

```java
package com.example.takahiro.tablayoutapplication;

import android.content.Context;
import android.support.v4.view.ViewPager;
import android.util.AttributeSet;
import android.view.MotionEvent;

public class HoldableViewPager extends ViewPager {

	// スワイプの禁止フラグ(true: スワイプ禁止, false: スワイプOK)
	boolean isSwipeHold = false;

	public void setSwipeHold(boolean enabled) {
		isSwipeHold = enabled;
	}
	
	// コンストラクタ
	public HoldableViewPager(Context context) {
		super(context);
	}

	// コンストラクタ
	public HoldableViewPager(Context context, AttributeSet attributeSet) {
		super(context, attributeSet);
	}

	@Override
	public boolean onTouchEvent(MotionEvent motionEvent) {
		if(isSwipeHold) {
			// スワイプ禁止の場合
			return false;
		}

		return super.onTouchEvent(motionEvent);
	}

	@Override
	public boolean onInterceptTouchEvent(MotionEvent event) {
		if(isSwipeHold) {
			// スワイプ禁止の場合
			return false;
		}

		return super.onInterceptTouchEvent(event);
	}
}
```

この`HoldableViewPager`を`MainActivity.java`で使います。  
使い方は簡単で`ViewPager`を`HoldableViewPager`で置き換えて、`isSwipeHold`をtrueにするだけです。  

```java
....

@Override
protected void onCreate(Bundle savedInstanceState) {
	....
	// xmlからTabLayoutの取得
	TabLayout tabLayout = (TabLayout) findViewById(R.id.tabs);
	// xmlからViewPagerを取得
	HoldableViewPager viewPager = (HoldableViewPager)findViewById(R.id.pager);
	// スワイプ禁止設定の追加
	viewPager.setSwipeHold(true);
	....
}
....
```

また、`content_main.xml`も`ViewPager`から`HoldableViewPager`に変更する必要があります。  

```java
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	...
	<com.example.takahiro.tablayoutapplication.HoldableViewPager
		android:id="@+id/pager"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:layout_below="@+id/tabs"
	 />
</RelativeLayout>
```

本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

