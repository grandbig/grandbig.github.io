---
layout: post
title: "Google Sign-In SDK for Androidを使ってみよう！"
date: 2015-11-08 12:20
comments: true
categories: Google android
---

####Google Sign-in SDKを使ったOAuth認証をやってみよう！
さて、本日はGoogle Sign-in SDKのAndroidでの使い方について見ていきたいと思います。  
Androidの場合、Google製のため、親和性は元々高いのですが、推奨されているGoogle Sign-in SDKを使うことでより快適なコーディングやUXを提供できるのではと期待しています。  

因みに、本書執筆時の環境は下記です。  
* 端末: Nexus6  
* OS: Android6.0  
* IDE: Android Studio 1.5 preview2  

では、早速、使い方を見ていきましょう。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

####Google Developers ConsoleでOAuth認証に必要な項目を登録しよう！
さて、実際にGoogle Sign-Inを始める前に、やらなければならないことがあります。  
それは[Google Developers Console](https://console.developers.google.com)での登録作業です。(iOSと同じです。)  

まず、OAuth2.0の同意画面を作成しましょう。  
因みに、筆者は既に登録済みのOAuth2.0クライアントIDがあったため、下図のように表示されます。  
**OAuth同意画面** というボタンをクリックして、OAuth同意画面の作成ページに移りましょう。  
![認証情報の確認ページ](/images/google-sign-in-android-0.png)  

OAuth同意画面では **メールアドレス** と **サービス名** を入力しましょう。(他は必要に応じて入力してください。)  
![OAuth2.0の同意画面を作成](/images/google-sign-in-android-1.png)  

OAuth同意画面が作成できたら、認証情報を追加しましょう。  
下図のように **OAuth2.0クライアントID** を選択してください。  
![OAuth2.0クライアントIDを作成](/images/google-sign-in-android-2.png)  

今回はAndroid用なので、下図の手順でクライアントIDを作成しましょう。  
![OAuth2.0クライアントISを作成2](/images/google-sign-in-android-3.png)  

これでクライアントIDが作成されたと思います。  
作成したクライアントIDを選択すれば、下図のような情報が得られるはずです。  
 ![Google Developer ConsoleのOAuth2.0 for iOS](/images/google-sign-in-android-6.png)  

####Google Sign-in SDK for Androidを実装しよう
[Googleの公式サイト](https://developers.google.com/identity/sign-in/android/?hl=ja)に書かれていますが、英語が苦手な方向け(+自分のメモ書き)として説明していきたいと思います。  

まずは、[Google Sign-In for Androidのガイドページ](https://developers.google.com/identity/sign-in/android/start?hl=ja)にアクセスしましょう。  
そうすると、順番に説明されていることがわかります。  
①はテストコードを手に入れるだけなので、自分のプロジェクトに組み込むなら②からはじめましょう。  
![GET A CONFIGURATION FILE](/images/google-sign-in-android-4.png)  

そうすると、下図のようなページに遷移します。  
![Enable Google services for your app](/images/google-sign-in-android-5.png)  

ここで、 **App name** にGoogle Developers Consoleのプロジェクト名を **『選択』** します。  
注意点として、App nameに記載されているデフォルト名を削除すると、選択可能なプロジェクト名が表示されます。  
App nameが入力した後に、 **Android package name** を入力しましょう。  

必要項目の入力が終わったら、Choose and configure servicesボタンを押しましょう。  
![Choose and configure servicesボタンを押す](/images/google-sign-in-android-7.png)  

結果、画面が遷移します。  
![Configuration Fileを作成しよう](/images/google-sign-in-android-8.png)  

画面下の方に **SHA1** を入力する項目があるので、これを入力して、 **ENABLE GOOGLE SIGN-IN** ボタンを選択しましょう。  
![Configuration Fileを作成しよう](/images/google-sign-in-android-9.png)  

因みにSHA1の調べ方は下記の通りです。  

1. ターミナルで/Users/{ユーザ名}/.androidの階層に移動  
2. `keytool -list -v -keystore "/Users/{ユーザ名}/.android/debug.keystore" -alias androiddebugkey -storepass android -keypass android`コマンドを入力  
3. 出力結果の証明書のフィンガプリント > SHA1の値を取得  

画面下の **Generate configuration files** ボタンが押せるようになるので、押しましょう。  
![Configuration Fileを作成しよう](/images/google-sign-in-android-10.png)  

jsonファイルのダウンロード画面が表示されるので、ダウンロードしましょう。  
![jsonファイルをダウンロード](/images/google-sign-in-android-11.png)  

必要なファイルが揃ったので、自身のプロジェクトのappフォルダ配下にダウンロードした`google-services.json`を配置しましょう。  

続いて、`build.gradle`ファイルの設定にGoogle Play Services Pluginを追加しましょう。  
まずは、`build.gradle(Project)`の`dependencies`内に`classpath 'com.google.gms:google-services:1.5.0-beta2'`を追加  
次に、`build.gradle(app)`の最上部に`apply plugin: 'com.google.gms.google-services'`を追加  
そして、`build.gradle(app)`の`dependencies`内に`compile 'com.google.android.gms:play-services-auth:8.3.0'`を追加  

これで用意が整いましたので、ソースを書いていきます。  

```java
// MainActivity.java
package ...
import android.content.Intent;
import android.os.Bundle;
import android.support.design.widget.FloatingActionButton;
import android.support.design.widget.Snackbar;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.Toolbar;
import android.util.Log;
import android.view.View;
import android.view.Menu;
import android.view.MenuItem;

import com.google.android.gms.auth.api.Auth;
import com.google.android.gms.auth.api.signin.GoogleSignInAccount;
import com.google.android.gms.auth.api.signin.GoogleSignInOptions;
import com.google.android.gms.auth.api.signin.GoogleSignInResult;
import com.google.android.gms.common.ConnectionResult;
import com.google.android.gms.common.SignInButton;
import com.google.android.gms.common.api.GoogleApiClient;

public class MainActivity extends AppCompatActivity implements
        GoogleApiClient.OnConnectionFailedListener, View.OnClickListener {
	
	private static final String TAG = "MainActivity";
	private static final int RC_SIGN_IN = 9001;
	private GoogleApiClient mGoogleApiClient;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);

		<省略>

		// Button listenersの設定
		findViewById(R.id.button_sign_in).setOnClickListener(this);

		// Google Sign-Inの設定
		GoogleSignInOptions gso = new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN).requestEmail().build();
		mGoogleApiClient = new GoogleApiClient.Builder(this)
				.enableAutoManage(this /* FragmentActivity */, this /* OnConnectionFailedListener */)
				.addApi(Auth.GOOGLE_SIGN_IN_API, gso)
				.build();

		// Google Sign-Inボタンの設定
		SignInButton signInButton = (SignInButton) findViewById(R.id.button_sign_in);
		signInButton.setSize(SignInButton.SIZE_STANDARD);
		signInButton.setScopes(gso.getScopeArray());
	}

	<省略>

	// Sign-Inした結果を受け取る処理
	@Override
	public void onActivityResult(int requestCode, int resultCode, Intent data) {
		super.onActivityResult(requestCode, resultCode, data);

		// Result returned from launching the Intent from GoogleSignInApi.getSignInIntent(...);
		if (requestCode == RC_SIGN_IN) {
			GoogleSignInResult result = Auth.GoogleSignInApi.getSignInResultFromIntent(data);
			handleSignInResult(result);
		}
	}

	// Sign-Inした結果を扱う処理
	private void handleSignInResult(GoogleSignInResult result) {
		Log.d(TAG, "handleSignInResult:" + result.isSuccess());
		if (result.isSuccess()) {
			// Signed in successfully, show authenticated UI.
			GoogleSignInAccount acct = result.getSignInAccount();
		} else {
			// Signed out, show unauthenticated UI.
		}
	}

	// Sign-In処理
	private void signIn() {
		Intent signInIntent = Auth.GoogleSignInApi.getSignInIntent(mGoogleApiClient);
		startActivityForResult(signInIntent, RC_SIGN_IN);
	}

	@Override
	public void onClick(View v) {
		switch (v.getId()) {
			case R.id.button_sign_in:
				signIn();
				break;
		}
	}
}
```

続いてレイアウトの作成  

```java
// content_main.xml
<com.google.android.gms.common.SignInButton
	android:id="@+id/button_sign_in"
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:layout_below="@+id/textView"
	android:layout_alignParentStart="true"
	android:layout_marginTop="100dp" />
```

これを実装すると下記のような画面遷移ができます。  

![アプリ起動後の画面](/images/google-sign-in-android-12.png)  
![アカウントの選択](/images/google-sign-in-android-13.png)  

Google Sign-In SDK for Androidを使うことで、UI/UXをiOS側と合わせることもできるので、非常に良いと思います。  
ぜひ使ってみてはいかがでしょうか。  

といったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

