title: Fresco使用教程(一)
date: 2015-04-03 13:02:04
categories:  Android
---


##摘要
Fresco是Facebook最新推出的一款用于Android应用中展示图片的强大图片库，可以从网络、本地存储和本地资源中加载图片。其中的Drawees可以显示占位符，直到图片加载完成。而当图片从屏幕上消失时，会自动释放内存。

<!--more-->

##快速开始
你可以通过Maven Central下载Fresco
- 通过Gradle

	dependencies {
	  compile 'com.facebook.fresco:fresco:0.1.0+'
	}

- 通过Maven

	<dependency>
	  <groupId>com.facebook.fresco</groupId>
	  <artifactId>fresco</artifactId>
	  <version>LATEST</version>
	</dependency>


##Fresco入门

如果你想下载显示一张图片，并且在加载过程中用占位图来显示它，就可以用SimpleDraweeView。

首先你想展示网络上的图片，需要在你的清单文件中声明联网权限

	  <uses-permission android:name="android.permission.INTERNET"/>

在app启动过程中，在你调用setContextView()之前要初始化Fresco这个类

	Fresco.initialize(context);

在Xml中，新增一个自定义的命名空间在最外层

	<!-- Any valid element will do here -->
	<LinearLayout 
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:fresco="http://schemas.android.com/apk/res-auto"
	    android:layout_height="match_parent"
	    android:layout_width="match_parent">

然后在布局中添加SimpleDraweeView控件

	<com.facebook.drawee.view.SimpleDraweeView
	    android:id="@+id/my_image_view"
	    android:layout_width="130dp"
	    android:layout_height="130dp"
	    fresco:placeholderImage="@drawable/my_drawable"
	  />

你只需要在代码中这样写就可以成功显示图片了。

	Uri uri = Uri.parse("http://frescolib.org/static/fresco-logo.png");
	SimpleDraweeView draweeView = (SimpleDraweeView) findViewById(R.id.my_image_view);
	draweeView.setImageURI(uri);

##URI的支持

| Type      |    Scheme     |     Fetch method used    |
| :-------- | --------: | :-------:          |
| File on network  | http://, https://    |  HttpURLConnection or network layer   |
| File on device   | file://                     | FileInputStream                                             |
| Content provider | content://         |ContentResolver|
| Asset in app     |	asset:/       | AssetManager|
| Resource in app  |res://                |Resources.openRawResource|

相信大家看完上面的QUICK START，已经可以简单的使用Fresco了，下篇来看一下Drawees的各种特性。