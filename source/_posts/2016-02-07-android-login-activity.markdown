---
layout: post
title: "Android Studioでの初回作成Activityから考えるログイン画面"
date: 2016-02-07 22:38
comments: true
categories: android java
---

####Androidアプリでのログイン画面を考えてみる
筆者はこれまでAndroidアプリでログイン画面を作成するとき、  

* Person Name / E-mail / Phone などの必須入力項目用のText Fields  
* Password / Password(Numeric) のパスワード入力用のText Fields  
* ログインするためのアクション用のButton  

のみを配置することを考えていました。  
今回はAndroid Studioで初回プロジェクト作成時に`Login Activity`を選択することで、  
Google側の意図を読み取ってみようと思います。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####Login画面のレイアウト
では、早速、Android Studioを起動して`Login Activity`を選択したプロジェクトを作成しましょう。  
(本ブログでは **Android Studio 2.0 Beta2** を利用しています。)  

プロジェクト作成時に`Add an Activity to Mobile`のところで`Login Activity`を選択してみましょう。  

![Login Activityを選択する](/images/login-activity-1.png)  

すると、下記のようなレイアウトファイルが作成されます。  

```java
// activity_login.xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:tools="http://schemas.android.com/tools"
	...
	tools:context="com.example.takahiro.loginapplicationsample.LoginActivity">

	<!-- Login progress -->
	<ProgressBar
		android:id="@+id/login_progress"
		style="?android:attr/progressBarStyleLarge"
		android:layout_width="wrap_content"
		android:layout_height="wrap_content"
		android:layout_marginBottom="8dp"
		android:visibility="gone" />

	<ScrollView
		android:id="@+id/login_form"
		android:layout_width="match_parent"
		android:layout_height="match_parent">

		<LinearLayout
			android:id="@+id/email_login_form"
			android:layout_width="match_parent"
			android:layout_height="wrap_content"
			android:orientation="vertical">

			<android.support.design.widget.TextInputLayout
				android:layout_width="match_parent"
				android:layout_height="wrap_content">

				<AutoCompleteTextView
					android:id="@+id/email"
					android:layout_width="match_parent"
					android:layout_height="wrap_content"
					android:hint="@string/prompt_email"
					android:inputType="textEmailAddress"
					android:maxLines="1"
					android:singleLine="true" />

			</android.support.design.widget.TextInputLayout>

			<android.support.design.widget.TextInputLayout
				android:layout_width="match_parent"
				android:layout_height="wrap_content">
			
				<EditText
					android:id="@+id/password"
					android:layout_width="match_parent"
					android:layout_height="wrap_content"
					android:hint="@string/prompt_password"
					android:imeActionId="@+id/login"
					android:imeActionLabel="@string/action_sign_in_short"
					android:imeOptions="actionUnspecified"
					android:inputType="textPassword"
					android:maxLines="1"
					android:singleLine="true" />

			</android.support.design.widget.TextInputLayout>
			
			<Button
				android:id="@+id/email_sign_in_button"
				style="?android:textAppearanceSmall"
				android:layout_width="match_parent"
				android:layout_height="wrap_content"
				android:layout_marginTop="16dp"
				android:text="@string/action_sign_in"
				android:textStyle="bold" />

		</LinearLayout>
	</ScrollView>
</LinearLayout>
```

実際に画面はこのようになっています。  
![ログイン画面](/images/login-activity-2.png)  

その成り立ちを見てみましょう。  

#####ProgressBarが配置されている
まず、目についたのは`ProgressBar`です。  
ログインボタンをタップした後に、サーバへの問い合わせをすることが通常だと思います。  
サーバ通信後、データを受け取るまでにはそれなりに時間がかかる可能性があります。  
なので、ユーザにロード中であることを知らせる`ProgressBar`は必要であるということなのでしょう。  

#####ScrollViewが配置されている
続いて、`ScrollView`に着目しました。  
デフォルトでは、EmailとPasswordに加えて、ログインボタンが設置されているのみですが、  
場合によってはユーザ名, 電話番号, 住所, クレジットカード番号などなど必要とされる項目が多くなる可能性があります。  
初めからそういったことを想定した作りになっているということでしょう。  

#####TextInputLayoutが配置されている
そして、何と言っても、`android.support.design.widget.TextInputLayout`は目立つのではないでしょうか？  
これは **Android Design Support Library** に含まれるマテリアルデザインを体現したパーツとなります。  
`TextView`等を囲むことで、ユーザの入力アクションに合わせて、入力項目タイトルがスムーズに移動するデザインを実装することができます。  
いわゆる[Floating Label](https://www.google.com/design/spec/components/text-fields.html#)というものですね。  

#####AutoCompleteTextViewが使われている
今回はEmailの入力が必要とのことで、`AutoCompleteTextView`によるユーザ補助機能がついています。  
もちろん、`AutoCompleteTextView`をレイアウト上で設定しても補助機能を実装することはできず、  
`AndroidManifest.xml`で設定が必要です。  

```java
// AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
	package="com.example.takahiro.loginapplicationsample">

	<!-- To auto-complete the email text field in the login form with the user's emails -->
	<uses-permission android:name="android.permission.GET_ACCOUNTS" />
	<uses-permission android:name="android.permission.READ_PROFILE" />
	<uses-permission android:name="android.permission.READ_CONTACTS" />

	...
</manifest>
```

このように`GET_ACCOUNTS`, `READ_PROFILE`, `READ_CONTACTS`の3つのパーミッションを設定します。  
これによって、下図のようにアプリ起動後に連絡先へのアクセス許可を求めるアラートを表示することができます。  

![連絡先へのアクセス許可アラート](/images/login-activity-3.png)  

####LoginActivityについて
続いて、`LoginActivity`はどのようになっているのか見ていきましょう。  

#####LoaderManagerを利用
`LoginActivity`では`LoaderManager`を利用しています。  

```java
// LoginActivity.java
public class LoginActivity extends AppCompatActivity implements LoaderCallbacks<Cursor> {

	...
	@Override
	public Loader<Cursor> onCreateLoader(int i, Bundle bundle) {
		return new CursorLoader(this,
			// Retrieve data rows for the device user's 'profile' contact.
			Uri.withAppendedPath(ContactsContract.Profile.CONTENT_URI,
				ContactsContract.Contacts.Data.CONTENT_DIRECTORY), ProfileQuery.PROJECTION,

			// Select only email addresses.
			ContactsContract.Contacts.Data.MIMETYPE +
				" = ?", new String[]{ContactsContract.CommonDataKinds.Email
			.CONTENT_ITEM_TYPE},

			// Show primary email addresses first. Note that there won't be
			// a primary email address if the user hasn't specified one.
			ContactsContract.Contacts.Data.IS_PRIMARY + " DESC");
	}
	
	@Override
	public void onLoadFinished(Loader<Cursor> cursorLoader, Cursor cursor) {
		List<String> emails = new ArrayList<>();
		cursor.moveToFirst();
		while (!cursor.isAfterLast()) {
			emails.add(cursor.getString(ProfileQuery.ADDRESS));
			cursor.moveToNext();
		}

		addEmailsToAutoComplete(emails);
	}

	@Override
	public void onLoaderReset(Loader<Cursor> cursorLoader) {
		
	}
	...
}
```

`LoaderCallbacks<Cursor>`を **implements** しているのは、  
`onCreateLoader`, `onLoadFinished`でEmailのオートコンプリート機能を実装するためです。  

#####showProgressメソッドの実装
レイアウトで実装した`ProgressBar`のアニメーションまで実装されています。  
また、AndroidのOSバージョンに合わせて適切な処理分けもされています。  

```java
// LoginActivity.java
@TargetApi(Build.VERSION_CODES.HONEYCOMB_MR2)
private void showProgress(final boolean show) {
	// On Honeycomb MR2 we have the ViewPropertyAnimator APIs, which allow
	// for very easy animations. If available, use these APIs to fade-in
	// the progress spinner.
	if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB_MR2) {
		int shortAnimTime = getResources().getInteger(android.R.integer.config_shortAnimTime);

		mLoginFormView.setVisibility(show ? View.GONE : View.VISIBLE);
		mLoginFormView.animate().setDuration(shortAnimTime).alpha(
				show ? 0 : 1).setListener(new AnimatorListenerAdapter() {
			@Override
			public void onAnimationEnd(Animator animation) {
				mLoginFormView.setVisibility(show ? View.GONE : View.VISIBLE);
			}
		});

		mProgressView.setVisibility(show ? View.VISIBLE : View.GONE);
		mProgressView.animate().setDuration(shortAnimTime).alpha(
				show ? 1 : 0).setListener(new AnimatorListenerAdapter() {
			@Override
			public void onAnimationEnd(Animator animation) {
				mProgressView.setVisibility(show ? View.VISIBLE : View.GONE);
			}
		});
	} else {
		// The ViewPropertyAnimator APIs are not available, so simply show
		// and hide the relevant UI components.
		mProgressView.setVisibility(show ? View.VISIBLE : View.GONE);
		mLoginFormView.setVisibility(show ? View.GONE : View.VISIBLE);
	}
}
```

#####非同期通信処理の実装
ログイン処理は通常、必ずサーバ通信を伴います。  
そのため、バックグラウンドでの通信処理が必要となるんですが、それが既に側だけ実装されています。  

```java
// LoginActivity.java
public class UserLoginTask extends AsyncTask<Void, Void, Boolean> {
	
	private final String mEmail;
	private final String mPassword;

	UserLoginTask(String email, String password) {
		mEmail = email;
		mPassword = password;
	}

	@Override
	protected Boolean doInBackground(Void... params) {
		// TODO: attempt authentication against a network service.
		
		try {
			// Simulate network access.
			Thread.sleep(2000);
		} catch (InterruptedException e) {
			return false;
		}

		for (String credential : DUMMY_CREDENTIALS) {
			String[] pieces = credential.split(":");
			if (pieces[0].equals(mEmail)) {
				// Account exists, return true if the password matches.
				return pieces[1].equals(mPassword);
			}
		}

		// TODO: register the new account here.
		return true;
	}
}
```

さて、いかがだったでしょうか？  
レイアウトはまだしも、オートコンプリートの実装まで必要な場合は、自作するよりも、Android Studioでデフォルト実装してくれる`LoginActivity`の方が工数省略ができそうですよね！
といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
