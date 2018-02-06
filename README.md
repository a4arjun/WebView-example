# WebView-example

In this tutorial, I'm going to tell you how to create a simple android webview app which has a splash screen, error page and an exit confirmation

In MainActivity,

```java

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.webkit.WebSettings;
import android.webkit.WebView;
import android.net.Uri;
import android.app.AlertDialog;
import android.content.DialogInterface;
public class MainActivity extends Activity {

    private WebView mWebView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mWebView = (WebView) findViewById(R.id.activity_main_webview);
        WebSettings webSettings = mWebView.getSettings();
        webSettings.setJavaScriptEnabled(true);
        mWebView.loadUrl("http://yoursitename.com");
        mWebView.setWebViewClient(new com.example.webview.WebViewClient(){
				@Override
				public void onPageFinished(WebView view, String url) {
					findViewById(R.id.progressBar1).setVisibility(View.GONE);
					findViewById(R.id.imageView).setVisibility(View.GONE);
					findViewById(R.id.activity_main_webview).setVisibility(View.VISIBLE);
				}

				public void onReceivedError(WebView view,int errorCode, String description, String failingUrl){
					mWebView.loadUrl("file:///android_asset/404.html");
				}
				public boolean shouldOverrideUrlLoading(WebView view, String url) {
					if (url.startsWith("tel:")) {
						Intent tel = new Intent(Intent.ACTION_DIAL, Uri.parse(url));
						startActivity(tel);
						return true;
					}
					else if (url.contains("mailto:")) {
						view.getContext().startActivity(
						new Intent(Intent.ACTION_VIEW, Uri.parse(url)));
						return true;

					}
					else if(url.equals("request://reload")){
						Intent intent=new Intent(getApplicationContext(),MainActivity.class);
						startActivity(intent);
						finish();
						return true;
					}
					else if(Uri.parse(url).getHost().endsWith("mykhd.info")) {
						return false;
					}
					Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
					view.getContext().startActivity(intent);
					return true;
				}


			});

    }


	@Override
	public void onBackPressed() {
		if (mWebView.canGoBack()) {
			mWebView.goBack();

		}
		else
		{
			AlertDialog.Builder builder = new AlertDialog.Builder(this);
			builder.setMessage("Are you sure you want to exit?")
					.setCancelable(false)
					.setPositiveButton("Yes", new DialogInterface.OnClickListener() {
						public void onClick(DialogInterface dialog, int id) {
							finish();
						}
					})
					.setNegativeButton("No", new DialogInterface.OnClickListener() {
						public void onClick(DialogInterface dialog, int id) {
							dialog.cancel();
						}
					});
			AlertDialog alert = builder.create();
			alert.show();
		}

	}

}
```
That's it.




Now we need to create a flash screen. Create new file as Splash.java. Use the code below.

```java

import android.annotation.SuppressLint;
import android.annotation.TargetApi;
import android.app.ActionBar;
import android.app.Activity;
import android.content.Intent;
import android.os.Build;
import android.os.Bundle;

@SuppressLint("NewApi")
public class Splash extends Activity {


    @TargetApi(Build.VERSION_CODES.HONEYCOMB)
    @SuppressLint("NewApi")
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_splash);

        ActionBar actionBar = getActionBar();
        actionBar.hide();

        Thread t =new Thread(){
            public void run(){
                try{
                    sleep(50000);
                }catch(InterruptedException e){
                    e.printStackTrace();
                }finally{
                    Intent i =new Intent(Splash.this,MainActivity.class);
                    startActivity(i);
                }
            }
        };
        t.start();
    }
    @Override
    public void onPause(){
        super.onPause();
        finish();
    }

}
```

We are done creating flash screen. 

Now, we have to create layout files. In your activity_main.xml, paste the code below.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
	xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:app="http://schemas.android.com/apk/res-auto"
	xmlns:tools="http://schemas.android.com/tools"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	tools:context=".MainActivity"
	android:orientation="vertical"
	android:background="#007afe">

	<ImageView
		android:id="@+id/imageView"
		android:layout_width="wrap_content"
		android:layout_height="360dp"
		android:layout_gravity="center"
		android:src="@drawable/splashscreen" />
	<ProgressBar
		android:id="@+id/progressBar1"
		android:layout_width="64dp"
		android:layout_height="wrap_content"
		android:layout_centerHorizontal="true"
		android:layout_gravity="center"
		android:indeterminate="true" />

	<WebView
		android:id="@+id/activity_main_webview"
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		android:visibility="gone"/>
    
</LinearLayout>

```


Now, we are ready to create a layout for splashscreen. For that, create a new layout file and name it activity_splash.xml. You can use the code below.
```xml
<RelativeLayout
	xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:tools="http://schemas.android.com/tools"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:background="#B7B3B3"
	tools:context=".Splash">

</RelativeLayout>
```
You need to place your splashscreen image in drawable folder. That image will be loaded when the app is started and closes after page loads.




We need an error page (404) to be loaded if there are no signs of internet connection. We are going to load the error page from android asset folder. Check MainActivity.jva to see how I linked to error page. in your error page, we need to add a reload or refresh button. for that, simply add this code in your 404 page.
```html
<a href="request://reload" class="btn">RELOAD</a>
```
By clicking the link, the MainActivity will be called and the page will be reloaded.




Now we are up to set our AndroidManifest file.

```xml
	<uses-permission
		android:name="android.permission.INTERNET"/>
    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/NoActionBar" >
        <activity
            android:name=".MainActivity"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
```
You have to add this code in AndroidManifest file. Now compile the app. You are done.


