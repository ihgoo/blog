title: 新的构建工具！Gradle的使用教程
date: 2015-03-10 10:08:05
categories: Android
tag: AndroidStudio
---
##Gradle
----
Gradle是以Groovy语言为基础，面向java应用为住。基于DSL（领域特定语言）语法的自动怕你规划构建工具。
gradle对多工程的构建支持很出色，工程依赖是gradle的第一公民。
gradle支持局部构建。
支持多方式以来管理：包括从maven远程仓库、nexus私服、ivy仓库以及本地文件系统的jars或者dirs
gradle是第一个构建继承工具，与ant、maven、ivy有良好的相关性
轻松迁移：gradle适用于任何结构的工程，你可以在同一个开发平台平行构建原工程和gradle工程。通常要求写相关测试，以保证开发的插件的相似醒呢，这种迁移可以减少破坏性，尽可能的可靠。这也是构建的最佳实践
gradle的整体设计是以作为一种语言为导向的，而非成为一个阉割死板的框架。
<!-- more -->
##Gradle构建Android程序
如果你是初次使用AndroidStudio，那么安装完成后将会让你选择配置离线的gradle文件目录还是重新从官网上下载gradle，在你国这网络的情况下，我们可以采用跳墙的方法来下载。(在AndroidStudio1.02之后就可以选择离线的gradle了，不需要每次第一次构建Android项目的时候都得下载gradle)。

![](http://ihgoo.qiniudn.com/gradle.png)

[**各个版本gradle的中转下载**](http://pan.baidu.com/s/1bnaZdOb)

下载好之后配置gradle环境变量，命令行输入gradle -v，查看成功配置与否。

![](http://ihgoo.qiniudn.com/gradle_v.png)


	在Gradle构建的Android项目中，与gradle相关的目录有以下几个：
	- build.gradle
		这个文件是app文件gradle配置文件，也可以算是整个项目最主要的配置文件，下面是内容：

	//声明是Android程序
	apply plugin: 'com.android.application'

	android {
	    //编译SDK的版本
	    compileSdkVersion 21
	    build tools的版本
	    buildToolsVersion "21.1.2"

	    defaultConfig {
	        //包名
	        applicationId "com.bqs.wetime.fruits"
	        minSdkVersion 14
	        targetSdkVersion 21
	        versionCode 1
	        versionName "1.0"
	    }
	    buildTypes {
	        release {
	        	//设置混淆
	            minifyEnabled false
	            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
	        }
	    }
	}

	// 依赖声明
	dependencies {
	//gradle可以直接拷贝jar到libs目录下，亦可用maven的方式来导入
	    compile 'com.squareup.retrofit:retrofit:1.9.0'
	    compile 'com.jakewharton:butterknife:6.1.0'
	    compile 'com.nostra13.universalimageloader:universal-image-loader:1.9.3'
	    compile 'com.squareup:otto:+'
	    compile 'com.android.support:appcompat-v7:21.0.3'
	    compile 'com.android.support:recyclerview-v7:21.0.3'
	    compile 'com.squareup.okhttp:okhttp:2.2.0'
	    compile 'com.google.code.gson:gson:2.3.1+'
	    compile 'de.greenrobot:java-common:2.0.0'
	    compile 'de.greenrobot:greendao:1.3.7'
	    compile 'com.github.traex.rippleeffect:library:1.2.3'
	    compile files('libs/rebound-0.3.7.jar')
	    compile files('libs/co.snappydb.jar')
	    compile files('libs/kryo-2.23-SNAPSHOT.jar')
	    compile project(':CoconutUtil')
	    compile 'com.android.support:recyclerview-v7:21.0.3'
	}

- gradle文件夹
- settings.gradle



##依赖声明
----
compile [group]:[name]:[version]
这是一种快捷的依赖方式
compile files 
对本地文件夹下的jar进行依赖
compile project
如果在这个项目中存在其他的module，就可以用这种方法。相当于eclipse中选中isLibrary选项框。


##build.gradle
----

	buildscript {
	    repositories {
	        jcenter()
	    }
	    dependencies {
	        classpath 'com.android.tools.build:gradle:1.0.0'

	        // NOTE: Do not place your application dependencies here; they belong
	        // in the individual module build.gradle files
	    }
	}

	allprojects {
	    repositories {
	        jcenter()
	    }
	}


jcenter是Gradle1.7之后采用的一种新的中央远程仓库，目前Github经常看到很多开源库将其上传至Jcenter仓库中，使用起来非常方便而且易于维护，尤其是在Android Sudio中，只需要几行代码的配置就可以上传上去。

##Gradle常用命令
----

* ./gradlew -v 版本号

* ./gradlew clean 清除9GAG/app目录下的build文件夹

* ./gradlew build 检查依赖并编译打包

这里注意的是 ./gradlew build 命令把debug、release环境的包都打出来，如果正式发布只需要打Release的包，该怎么办呢，下面介绍一个很有用的命令 assemble, 如

* ./gradlew assembleDebug 编译并打Debug包

* ./gradlew assembleRelease 编译并打Release的包

除此之外，assemble还可以和productFlavors结合使用，具体在下一篇多渠道打包进一步解释。

* ./gradlew installRelease Release模式打包并安装

* ./gradlew uninstallRelease 卸载Release模式包



本文参考于：

[Android Studio系列教程五--Gradle命令详解与导入第三方包](http://segmentfault.com/blog/stormzhang/1190000002464822)

[Gradle介绍引用自百度百科](http://baike.baidu.com/link?url=ow-t1eOvCniKu9H5PygoaZgNYBEk2JPUsKzwVQncNF045xdTGq6KMcb-hMW_ZAQEBroM3lfZ0e5BuYPOFDXyd_)