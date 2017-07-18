WebView

1 常见Webview的方法

加载页面：  
//方式1. 加载一个网页：
webView.loadUrl("http://www.google.com/");

//方式2：加载apk包中的html页面
webView.loadUrl("file:///android_asset/test.html");

// 方式4： 加载 HTML 页面的一小段内容
WebView.loadData(String data, String mimeType, String encoding)
// 参数说明：
// 参数1：需要截取展示的内容
// 内容里不能出现 ’#’, ‘%’, ‘\’ , ‘?’ 这四个字符，若出现了需用 %23, %25, %27, %3f 对应来替代，否则会出现异常
// 参数2：展示内容的类型
// 参数3：字节码

控件的状态:
//激活WebView为活跃状态，能正常执行网页的响应
webView.onResume() ；

//当页面被失去焦点被切换到后台不可见状态，需要执行onPause
//通过onPause动作通知内核暂停所有的动作，比如DOM的解析、plugin的执行、JavaScript执行。
webView.onPause()；

//当应用程序(存在webview)被切换到后台时，这个方法不仅仅针对当前的webview而是全局的全应用程序的webview
//它会暂停所有webview的layout，parsing，javascripttimer。降低CPU功耗。
webView.pauseTimers()
//恢复pauseTimers状态
webView.resumeTimers()；

//销毁Webview
//在关闭了Activity时，如果Webview的音乐或视频，还在播放。就必须销毁Webview
//但是注意：webview调用destory时,webview仍绑定在Activity上
//这是由于自定义webview构建时传入了该Activity的context对象
//因此需要先从父容器中移除webview,然后再销毁webview:
rootLayout.removeView(webView); 
webView.destroy();

关于前进后退：
//是否可以后退
Webview.canGoBack() 
//后退网页
Webview.goBack()
//通常可以重新onPressBack来实现webview的后退操作

//是否可以前进                     
Webview.canGoForward()
//前进网页
Webview.goForward()

//以当前的index为起始点前进或者后退到历史记录中指定的steps
//如果steps为负数则为后退，正数则为前进
Webview.goBackOrForward(intsteps) 

2 WebSetting 对WebView进行配置和管理
//声明WebSettings子类
WebSettings webSettings = webView.getSettings();

//如果访问的页面中要与Javascript交互，则webview必须设置支持Javascript
webSettings.setJavaScriptEnabled(true);  
// 若加载的 html 里有JS 在执行动画等操作，会造成资源浪费（CPU、电量）
// 在 onStop 和 onResume 里分别把 setJavaScriptEnabled() 给设置成 false 和 true 即可

//支持插件
webSettings.setPluginsEnabled(true); 

//设置自适应屏幕，两者合用
webSettings.setUseWideViewPort(true); //将图片调整到适合webview的大小 
webSettings.setLoadWithOverviewMode(true); // 缩放至屏幕的大小

//缩放操作
webSettings.setSupportZoom(true); //支持缩放，默认为true。是下面那个的前提。
webSettings.setBuiltInZoomControls(true); //设置内置的缩放控件。若为false，则该WebView不可缩放
webSettings.setDisplayZoomControls(false); //隐藏原生的缩放控件

//其他细节操作
webSettings.setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK); //设置缓存策略
webSettings.setAllowFileAccess(true); //设置可以访问文件 
webSettings.setJavaScriptCanOpenWindowsAutomatically(true); //支持通过JS打开新窗口 
webSettings.setLoadsImagesAutomatically(true); //支持自动加载图片
webSettings.setDefaultTextEncodingName("utf-8");//设置编码格式

通常的缓存策略：
//优先使用缓存: 
WebView.getSettings().setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK); 
  //缓存模式如下：
  //LOAD_CACHE_ONLY: 不使用网络，只读取本地缓存数据
  //LOAD_DEFAULT: （默认）根据cache-control决定是否从网络上取数据。
  //LOAD_NO_CACHE: 不使用缓存，只从网络获取数据.
  //LOAD_CACHE_ELSE_NETWORK，只要本地有，无论是否过期，或者no-cache，都使用缓存中的数据。

//不使用缓存: 
WebView.getSettings().setCacheMode(WebSettings.LOAD_NO_CACHE);

3.WebViewClient类
作用：处理各种通知 & 请求事件
shouldOverrideUrlLoading()

作用：打开网页时不调用系统浏览器， 而是在本WebView中显示；在网页上的所有加载都经过这个方法,这个函数我们可以做很多操作。
webView.setWebViewClient(new WebViewClient(){
@Override
public boolean shouldOverrideUrlLoading(WebView view, String url) {
  view.loadUrl(url);
return true;
}

onPageStarted()
作用：开始载入页面调用的，我们可以设定一个loading的页面，告诉用户程序在等待网络响应。请注明出处。

onPageStarted()
作用：开始载入页面调用的，我们可以设定一个loading的页面，告诉用户程序在等待网络响应。

onLoadResource()
作用：在加载页面资源时会调用，每一个资源（比如图片）的加载都会调用一次。

4.WebChromeClient类
作用：辅助 WebView 处理 Javascript 的对话框,网站图标,网站标题等等。

onProgressChanged（）	
作用：获得网页的加载进度并显示

onReceivedTitle（）
作用：获取Web页中的标题

onJsAlert（） onJsConfirm（） onJsPrompt（）这几个都是js函数的调用

Android WebView自带的缓存机制有5种：
浏览器 缓存机制   
这种是根据http协议Cache-Control字段来控制的文件的缓存机制 根据缓存的有效时间，超过有效时间则重新请求，这个是由webview自动来进行的，无需操作

Application Cache 缓存机制
// 通过设置WebView的settings来实现
WebSettings settings = getSettings();

String cacheDirPath = context.getFilesDir().getAbsolutePath()+"cache/";
settings.setAppCachePath(cacheDirPath);
// 1. 设置缓存路径

 settings.setAppCacheMaxSize(20*1024*1024);
// 2. 设置缓存大小

settings.setAppCacheEnabled(true);
// 3. 开启Application Cache存储机制

Dom Storage 缓存机制
Web SQL Database 缓存机制
Indexed Database 缓存机制
//打开js交互这个就默认打开了

js与Android的交互
1. Android通过WebView调用 JS 代码
两种方法
	1.通过WebView的loadUrl（）  2.通过WebView的evaluateJavascript（）
	eg：
	mWebView.loadUrl("file:///android_asset/javascript.html"); //载入h5文件

	//h5文件中包含的js
	<script>
	// Android需要调用的方法
	   function callJS(){
	      alert("Android调用了JS的callJS方法");
	   }
	</script>

	// 设置与Js交互的权限
    webSettings.setJavaScriptEnabled(true);		
	mWebView.loadUrl("javascript:callJS()"); //注意这里要写javascript4
	特别注意：JS代码调用一定要在 onPageFinished（） 回调之后才能调用，否则不会调用。

	2.通过WebView的evaluateJavascript（）
	evaluateJavascript(String script, ValueCallback<String> resultCallback)
	script 为对应的方法  resultCallback 为返回值

2.JS通过WebView调用Android
三种方法
	1.通过WebView的addJavascriptInterface（）进行对象映射
	mWebView.addJavascriptInterface(new AndroidtoJs(), "test");//AndroidtoJS类对象映射到js的test对象

	//定义Android对象
	public class AndroidtoJs extends Object {
	    // 定义JS需要调用的方法
	    // 被JS调用的方法必须加入@JavascriptInterface注解
	    @JavascriptInterface
	    public void hello(String msg) {
	        System.out.println("JS调用了Android的hello方法");
	    }
	}

	//JS调用如下
	<script>
	     function callAndroid(){
	    // 由于对象映射，所以调用test对象等于调用Android映射的对象
	        test.hello("js调用了android中的hello方法");
	     }
	 </script>


	2.通过 WebViewClient 的shouldOverrideUrlLoading ()方法回调拦截 url

	3.通过 WebChromeClient 的onJsAlert()、onJsConfirm()、onJsPrompt（）方法回调拦截JS对话框alert()、confirm()、prompt（） 消息
	Android通过WebChromeClient复写onJsPrompt（）等函数来实现在android中的调用


