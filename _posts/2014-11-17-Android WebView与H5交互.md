---
layout: post
title:  "Android WebView与H5交互!"
date:   2017-01-17 13:31:01 +0800
categories: jekyll
tag: jekyll
---

* content
{:toc}


First POST build by Jekyll.
github:https://github.com/nishibaiyang


Android WebView与H5交互
------------------------

## 挖掘容器的最大潜力：配置合适的Webview属性 ##
 Android中的WebView组件，在4.4以前的版本是WebKit的内核，4.4以后换成chromium的内核，可以直接显示和渲染web页面，直接显示网页，也可以直接用html文件（网络上或本地assets中）作布局和JavaScript交互调用。Webview提供了很多设置项，在App内嵌H5时，要结合自己H5的特点去选取合适的设置项进行配置。而且，当你发现某些H5的特性没有体现出来、没有加载成功的时候，也可以先去查一查是不是还需要设置Webview的哪些属性。经常用到的配置包括以下：
     webSettings.setJavaScriptEnabled(true);//支持js

        webSettings.setJavaScriptCanOpenWindowsAutomatically(true);//支持js打开新窗口

        webSettings.setUseWideViewPort(true); //设置webview可任意比例缩放

        webSettings.setLoadWithOverviewMode(true);// 缩放至屏幕的大小

        webSettings.setRenderPriority(WebSettings.RenderPriority.HIGH);//提高渲染的优先级

        webSettings.setSaveFormData(true);// 保存表单数据

        webSettings.setDomStorageEnabled(true); // 开启 DOM storage API 功能

        webSettings.setUserAgentString(APIRequestResult.getUserAgent());//自定义UserAgent

        webSettings.setAllowFileAccess(true);// 允许访问文件

        webSettings.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);

        //http、https混合访问

## 按需改造：让Webview与H5更和谐 ##
目前H5页面的功能越来越丰富，从简单的弹框提示、输入确认，到各种爆款H5中的上传图片分析、视频播放等等，这些功能都非常常见，但是这些功能都一定程度上需要原生支持或者可以在原生Webview侧加以改造把体验优化的更好。
### 改造必备项：在当前的webview中跳转到新的url ###
Webview在当前页面打开页面内的跳转，而不是调起浏览器或者打开新页面：重写WebViewClient中的shouldOverrideUrlLoading方法。这个方法其实还有更多的作用，H5的js代码与原生交互也会用到。
    webView.setWebViewClient(new WebViewClient() {
public boolean shouldOverrideUrlLoading(WebView view, String url) {
               view.loadUrl(url); //在当前的webview中跳转到新的url
               return true;
        }
});
#### alert弹框优化 ####
在前端开发中，alert弹框很常见。但是在Webview里面，弹出来的默认弹框就会很丑，而且标题往往是“来自file://.....”或者“网址“xxx”的网页显示”,很不和谐，需要将这样的弹框改造成和原生App风格一致的弹框保证体验。重写WebChromeClient中的onJsAlert、onJsConfirm、onJsPrompt方法，对应改造提示弹框、确认弹框、需要提交文本的弹框，可以在方法中使用原生自定义弹框进行提示。
#### 支持H5下载文件 ####
 当碰到页面有下载链接的时候，比如H5页面提示用户更新安装包，默认情况下点击上去是一点反应都没有的。原来是因为WebView默认没有开启文件下载的功能，如果要实现文件下载的功能，需要设置WebView的DownloadListener，通过原生代码实现自己的DownloadListener来进行文件的下载。
    webView.setDownloadListener(new DownloadListener(){
@Override
    public void onDownloadStart(String url, String userAgent, String contentDisposition, String mimetype,
                                long contentLength) {
        Uri uri = Uri.parse(url);
        Intent intent = new Intent(Intent.ACTION_VIEW, uri);
        startActivity(intent);
    }
});
#### 支持H5选择文件或拍照功能 ####
现在H5页面经常需要实现上传头像的功能，默认的Webview也是不支持的。当网页需要打开手机相机拍照或者选择本地图片文件时，需要重写WebViewClient的openFileChooser相关的方法。当我们在Web页面上点击选择文件的控件(<input type="file">)时，会回调WebChromeClient下的openFileChooser()（5.0及以上系统回调onShowFileChooser()）。这个时候我们在openFileChooser方法中通过Intent打开系统相册或者支持该Intent的第三方应用来选择图片：
    // For Android 4.1
public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType, String capture) {
mUploadMessage = uploadMsg;
    Intent i = new Intent(Intent.ACTION_GET_CONTENT);
    i.addCategory(Intent.CATEGORY_OPENABLE);
    i.setType("image/*");
    WebviewActivity.this.startActivityForResult(Intent.createChooser(i, "File Chooser"),
            WebviewActivity.FILECHOOSER_RESULTCODE);
}

最后我们在onActivityResult()中将选择的图片内容通过ValueCallback的onReceiveValue方法返回给WebView。

        但是，就是这个openFileChooser方法其实有很多坑，在Android不同版本上的参数、使用方法甚至方法名称都不一样，需要一个个分别适配。Google的大神们也是用心良苦，对openFileChooser做了多次修改，在5.0上更是将回调方法该为了onShowFileChooser。所以为了解决这一问题，兼容各个版本，我们需要对openFileChooser()进行重载，同时针对5.0及以上系统提供onShowFileChooser()方法。
#### 自定义出错页面 ####
当WebView加载页面出错时（一般为404 NOT FOUND），安卓WebView会默认显示一个比较挫的出错界面。当然我们肯定不希望用户发现原来App里面竟然是个出错的网页啊，我们要原生的和谐的体验的啊，那就需要干掉这个默认出错页面。当WebView加载出错时，我们会在WebViewClient实例中的onReceivedError()方法接收到错误，我们就在这里做些手脚：

public void onReceivedError(WebView view, int errorCode,
                            String description, String failingUrl) { 
    ((TextView) findViewById(R.id.webViewerror_tv)).setText("点击刷新");
    view.loadDataWithBaseURL(null, "", "text/html", "utf-8", null);
    mRelativeLayout.setVisibility(View.VISIBLE);
    mRelativeLayout.setOnClickListener(new View.OnClickListener() {
@Override
        public void onClick(View view) {
            refreshWebview();
        }
    });
}
        从上面可以看出，我们先使用loadDataWithBaseURL清除掉默认错误页内容，再让我们自定义的View得到显示得到原生自定义的UI效果，并且加上点击刷新页面事件。
####　支持H5视频全屏播放　####
现在支持视频、音乐基本上成为一个爆款H5的标配，这同样需要原生支持。主要思路是，在页面级打开硬件加速功能：在AndroidManifest.xml中声明HardwareAccelerate的标志，一般是添加在Activity的级别上。代码如下：

                                         <activity ... android:hardwareAccelerated="true" >

        重写WebChromeClient的onShowCustomView及onHideCustomView接口，将Webview替换成一个临时的父View。进入全屏时会触发onShowCustomView回调方法，在onShowCustomView方法中，将获取到的view放到当前Activity的最上方，在onHideCustomView中，将之前的view隐藏或者删除，将原来被覆盖的webview放回来。

## Js与原生交互的方法 ##
由于原生端调用网页端js可以通过WebView.loadUrl(“javascript:function()”)的方式很便捷的实现，因 此Js与原生交互的难点就在js如何调用原生。Android webview web页面js调用原生代码目前有四种方式：jsBridge、prompt、console.log、addJavascriptInterface。从本质上来说，就是两种方式：一种是通过制定一些scheme协议，即前端与终端事先约定好调用的协议，匹配请求并拦截处理，jsBridge、prompt、console.log都是这种方式，另一种就是给js提供一个真实的对象，供其注入接口，addJavascriptInterface就是Android系统Webview本身提供的能力。
### jsBridge方式 ###
  jsBridge方式简单来说就是利用上文提到过的，页面出现跳转请求时，会调用WebViewClient的shouldOverrideUrlLoading方法加载URL，原生与H5可以规定请求的伪协议，并在此方法中对URL进行拦截，若当前请求协议与约定协议成功匹配，则拦截跳转，调用协议对应的接口。
###  prompt方式 ###
 prompt方式也是通过上文提到过的弹框优化中重写WebChromeClient的onJsPrompt方法，可以拦截window对象的prompt方法，onJsPrompt方法通过参数可以获取到defaultValue参数，该参数可以设为与native层调用的伪协议，从而进行拦截处理。
### console.log方式 ###
console.log方式则是通过设置webview的WebChromeClient重写onConsoleMessage方法可以拦截console对象的log方法，onConsoleMessage通过参数可以获取到message参数，该参数可以设为与native层调用的伪协议，从而进行拦截处理。
### addJavascriptInterface ###
对于addJavascriptInterface方法，使用起来也很方便，但是由于Java有反射机制以及Android系统前期的不成熟，会有一些安全相关的问题要注意：
Android 3.0以上版本，需要移除风险接口

                    mWebView.removeJavascriptInterface("searchBoxJavaBridge_");

                    mWebView.removeJavascriptInterface("accessibility");

                    mWebView.removeJavascriptInterface("accessibilityTraversal");

        Android4.2及以上的APP中，JS只能访问带有 @JavascriptInterface注解的Java函数，安全性得到了保证。这里不比较四种方法具体性能，但是jsBridge和addJavascriptInterface两种方法是使用的非常普遍的。

## 需要提供哪些原生接口 ##
 首先我们没必要像小程序那样做得那么通用，形成一套自己的生态系统，我们的目的是要让我们自己的App中内嵌的H5能够体验尽可能的流畅而不额外增加特别多的工作量，因此我们可以根据实际情况去实现一些必要的或者对体验改进有提高的接口。可以简单分类如下：

         UI相关接口：全屏Loading、局部Loading、TOP栏按钮、弹框组件

         登录态相关接口：切换登陆、刷新登录态、登录态失效跳转、跳转登录页面

         页面跳转相关接口：打开原生特定页面

        特别是在UI组件层面上，提供性能优秀的的原生组件模块对内嵌H5的体验将会有更大的提升。
## 减少H5加载的等待时间 ##
在H5页面载入时，由于网络原因、接口原因或者页面渲染原因，可能并不总能和原生一样很快的加载出来。因此我们往往会做一个小阔比较酷炫的loading页面来掩盖页面还没加载出来时的空白屏。但是这个loading页面什么时候该消失呢？

        WebView的WebViewClient实例中提供了onPageFinished()方法来监听页面是否加载完成，但是这个方法并不一定真正的及时执行。你永远无法确定当WebView调用这个方法的时候，网页内容是否真的加载完毕了，并且当前正在加载的网页产生跳转的时候这个方法可能会被多次调用。因此依赖这个方法来取消loading并不是最优的选择。

        但是对于H5页面来说，js执行到哪里了、执行完没有，页面框架是否已经加载出来了，哪些资源可能加载的耗时较长这相对来说是比较清楚的，因此我们可以给js提供下面的原生接口：

                                               showHUD()  显示全屏的loading

                                               hideHUD()   隐藏全屏的loading

                                               showLoading()    显示局部的loading

                                               hideLoading()"   隐藏局部的loading

        当页面开始加载的时候，原生会显示全屏的loading，当页面框架加载完成时，js调用hideHUD让loading页消失，减小用户等待时间，如果此时还有部分页面资源没加载完、或者需要刷新，则调用局部loading的原生方法提示用户，改进体验。WebView的onPageFinished方法也可以调用hideHUD，保证谁更快监听到加载完成事件都能更快让用户看到页面。
## Cookie和UserAgent ##
 Webview登录态可以注入在cookie中，通过CookieManager中的：setCookie、getCookie、removeAllCookie方法设置。以微信登录为例，可以在cookie中保存openid和access_token作为用户的登录态。原生提供如下接口给js调用：

                                                refreshTicket()   刷新登录票据       

                                                changeAccount()    切换账号

                                                showLogin()     拉起登录界面

        当页面接口校验登录态失效时，可以用refreshTicket接口向原生SDK提供的方法进行登录态续期，App都多账号体系时也可以通过changeAccount切换到其他的登录态，并且可以通过showLogin接口跳转到原生的登录页面，尽可能多的给用户原生的体验。

        UserAgent：一般来说App对应的服务端程序或者H5页面，会根据客户端传来的UserAgent来做不同的事情，比如系统版本、分辨率等等，可以自定义自己的App的UserAgent提供更多个性化的功能。使用getDefaultUserAgent()方法可以获取默认的UserAgent，也可以通过：mWebView.getSettings().setUserAgentString(ua);mWebView.getSettings().getUserAgentString();来设置和获取自定义的UserAgent。

# WebView的内存泄露 #
 根据实际项目中的测试和crash上报，webview导致的内存泄露的情况还是很严重的，尤其是当你加载的页面比较庞大的时候，可以通过以下方式尽量规避：

       在页面destory的时候，需要解除webview与activity的依赖，调用WebView.onPause()以及WebView.destory()，以便让系统释放WebView相关资源销毁webview，防止内存泄漏：
无法释放js导致耗电
        在有的手机里，如果webview加载的html中有一些js一直在执行比如动画之类的东西如果此刻webview 挂在了后台，这些资源是不会被释放用户也无法感知，导致一直占有cpu 耗电特别快，可以在页面的onStop()和onResume()里分别把setJavaScriptEnabled();给设置成false和true。
        import android.content.Context;
import android.graphics.Bitmap;
import android.net.ConnectivityManager;
import android.net.NetworkInfo;
import android.os.Build;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.Toolbar;
import android.text.TextUtils;
import android.util.Log;
import android.view.KeyEvent;
import android.view.View;
import android.view.Window;
import android.webkit.JsResult;
import android.webkit.WebChromeClient;
import android.webkit.WebResourceError;
import android.webkit.WebResourceRequest;
import android.webkit.WebSettings;
import android.webkit.WebView;
import android.webkit.WebViewClient;
import android.widget.ProgressBar;

import com.jockeyjs.Jockey;
import com.jockeyjs.JockeyImpl;

public class H5Activity extends AppCompatActivity {

    public static final String H5_URL = "H5_URL";
    private static final String JOCKEY_EVENT_NAME = "JOCKEY_EVENT_NAME";
    private static final String TAG = H5Activity.class.getSimpleName();

    private Toolbar mToolbar;
    private ProgressBar mProgressBar;

    private Jockey mJockey;
    private WebView mWebView;
    private WebViewClient mWebViewClient;
    private WebChromeClient mWebChromeClient;

    private String mUrl;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        supportRequestWindowFeature(Window.FEATURE_NO_TITLE);
        setContentView(R.layout.activity_h5);
        setupView();
        setupSettings();
    }

    @Override
    protected void onStart() {
        super.onStart();
        setupJockey();
        setupData();
    }

    private void setupView() {
        mToolbar = (Toolbar) findViewById(R.id.h5_toolbar);
        mProgressBar = (ProgressBar) findViewById(R.id.h5_progressbar);
        mWebView = (WebView) findViewById(R.id.h5_webview);
    }

    private void setupSettings() {

        mWebView.setScrollBarStyle(WebView.SCROLLBARS_INSIDE_OVERLAY);
        mWebView.setHorizontalScrollBarEnabled(false);
        mWebView.setOverScrollMode(WebView.OVER_SCROLL_NEVER);

        WebSettings mWebSettings = mWebView.getSettings();
        mWebSettings.setSupportZoom(true);
        mWebSettings.setLoadWithOverviewMode(true);
        mWebSettings.setUseWideViewPort(true);
        mWebSettings.setDefaultTextEncodingName("utf-8");
        mWebSettings.setLoadsImagesAutomatically(true);

        //JS
        mWebSettings.setJavaScriptEnabled(true);
        mWebSettings.setJavaScriptCanOpenWindowsAutomatically(true);

        mWebSettings.setAllowFileAccess(true);
        mWebSettings.setUseWideViewPort(true);
        mWebSettings.setDatabaseEnabled(true);
        mWebSettings.setLoadWithOverviewMode(true);
        mWebSettings.setDomStorageEnabled(true);


        //缓存
        ConnectivityManager connectivityManager = (ConnectivityManager) this.getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo info = connectivityManager.getActiveNetworkInfo();
        if (info != null && info.isConnected()) {
            String wvcc = info.getTypeName();
            Log.d(TAG, "current network: " + wvcc);
            mWebSettings.setCacheMode(WebSettings.LOAD_DEFAULT);
        } else {
            Log.d(TAG, "No network is connected, use cache");
            mWebSettings.setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK);
        }

        if (Build.VERSION.SDK_INT >= 16) {
            mWebSettings.setAllowFileAccessFromFileURLs(true);
            mWebSettings.setAllowUniversalAccessFromFileURLs(true);
        }

        if (Build.VERSION.SDK_INT >= 12) {
            mWebSettings.setAllowContentAccess(true);
        }

        setupWebViewClient();
        setupWebChromeClient();
    }

    private void setupJockey() {
        mJockey = JockeyImpl.getDefault();
        mJockey.configure(mWebView);
        mJockey.setWebViewClient(mWebViewClient);
        mJockey.setOnValidateListener(new Jockey.OnValidateListener() {
            @Override
            public boolean validate(String host) {
                return "yourdomain.com".equals(host);
            }
        });

        //TODO set your event handler
        mJockey.on(JOCKEY_EVENT_NAME, new EventHandler());
    }

    private void setupData() {
        mUrl = getIntent().getStringExtra(H5_URL);
        if (TextUtils.isEmpty(mUrl)) {
            //TODO show error page
        } else {
            mWebView.loadUrl(mUrl);
        }
    }

    private void setupWebViewClient() {
        mWebViewClient = new WebViewClient() {
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
                //TODO 处理URL, 例如对指定的URL做不同的处理等
                return false;
            }

            @Override
            public void onPageFinished(WebView view, String url) {
                super.onPageFinished(view, url);
            }

            @Override
            public void onPageStarted(WebView view, String url, Bitmap favicon) {
                super.onPageStarted(view, url, favicon);
            }

            @Override
            public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) {
                super.onReceivedError(view, request, error);
            }
        };
        mWebView.setWebViewClient(mWebViewClient);
    }

    private void setupWebChromeClient() {
        mWebChromeClient = new WebChromeClient() {
            @Override
            public void onReceivedTitle(WebView view, String title) {
                super.onReceivedTitle(view, title);
                mToolbar.setTitle(title);

            }

            @Override
            public void onProgressChanged(WebView view, int newProgress) {
                super.onProgressChanged(view, newProgress);
                mProgressBar.setProgress(newProgress);
                if (newProgress == 100) {
                    mProgressBar.setVisibility(View.GONE);
                } else {
                    mProgressBar.setVisibility(View.VISIBLE);
                }
            }

            @Override
            public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
                return super.onJsAlert(view, url, message, result);
            }
        };
        mWebView.setWebChromeClient(mWebChromeClient);
    }

    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if ((keyCode == KeyEvent.KEYCODE_BACK) && mWebView.canGoBack()) {
            mWebView.goBack();
            return true;
        }
        return super.onKeyDown(keyCode, event);
    }
}


[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
