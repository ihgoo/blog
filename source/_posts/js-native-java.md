title: Android中WebView无缝调用native代码，js和Java代码互调
date: 2015-03-06 10:40:57
categories: Android
---
webview是Android中最强大的控件之一，webview无缝打通了java和javascript的调用。
最近在开发之中发现html5+javascript实现效果比较简单，而且耗时和性能消耗也不算太多，H5适配起来也非常简单。

在webview中调用java扫描二维码并识别的方法，回传至服务器，或者是在android上做报表，饼状图、曲线图、柱状图，js有很成熟好用的绘图库。

<!-- more -->

##java调用js函数
------
java调用js函数非常简单，首先启用webview的Javascript支持。


	mWebView.getSettings().setJavaScriptEnabled(true);


再增加一个js函数reload(),函数仅供参考，函数内代码可根据业务需求写。


	    function reload(){
	         var per=$('#per').val(),
	         size=$('#size').val();
	         $('#circle').circleProgress({
	            value: 0.77*per,
	            size: size,
	            fill: {
	                image: "images/circle1.png"
	            },
	            startAngle: -Math.PI*1.265
	        });
	    }

在java中调用


	mWebView.loadUrl("javascript:"+reload+"()");


如果想调用JS中的有参函数，则 mWebView.loadUrl("javascript:"+functionName+"('"+arg+"')");   这样就可以了。


##JS调用Java代码
------
注册在JS中调用的对象名npc

	mWebView.getSettings().setJavaScriptEnabled(true);
	mWebView.addJavascriptInterface(this,"npc");

写一个Java方法，如果要兼容Android4.2+的版本@JavascriptInterface注解是必须加的，因为安全问题，在Android4.2中(如果应用的android:targetSdkVersion数值为17+)JS只能访问带有@JavascriptInterface注解的Java函数。

	    @JavascriptInterface
	    public void callCalculator(){
	        Intent intent = new Intent(getActivity(),CalculatorActivity.class);
	        startActivity(intent);
	    }

在JS中可以调用这个Java的方法了

	window.npc.callCalculator();

##H5的适配
Html的适配需要用js调用java的代码获取屏幕的宽或者高(像素)，获取到了宽高适配起来就比较简单了，在css中根据比例缩放就可以了。

另外还有能js调用java代码就用js调用java，因为java调用js代码耗时比js调用java代码要耗时的多。