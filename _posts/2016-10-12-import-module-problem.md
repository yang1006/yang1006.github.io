---
layout:     post
title:      "ANDROID 导入第三方模块一些常见的问题"
date:       2016-10-20 10:20:10 +0800
categories: Android
tags:       Android module
author: yang1006
---
* content
{:toc}

记录自己平时在Android开发时导入第三方模块一些常见的问题。





在开发中，我们经常会导入一些第三方的模块作为我们app的子模块使用，这里总结了一些导入第三方模块会经常遇见的问题：

1. 导入时使用File->New->Import Module导入，导入后为自己的项目添加模块依赖(Module Dependency)将模块依赖进来
2. 如果遇到子module的manifest中也有声明application,这时编译会报错，需要在自己项目manifest的application标签下增加:

		tools:replace="android:name"
		tools:overrideLibrary:"com.example.own"//com.example.own是导入项目的包名
3. 两个manifest中有的meta-data会有相同的name，需要在主模块对应的meta-data标签下增加：

		tools:replace=”android:value”
4. 两个项目的minSdkVersion需要保持一致，或者在主模块的uses-sdk标签下添加：

		tools:overrideLibrary="com.example.own" // com.example.own是导入项目的包名
5. 编译报错，提示java.exe返回值不是0而是1,2之类的，说明两个项目中有jar包重复了，需要删除重复的jar包(不同jar包中也可能有重复的类，需要用压缩软件打开jar文件删除所有重复的类)

6. 编译无问题后运行，发现导入的项目运行不起来，调试发现导入项目也有一个继承Application的类，由于一个apk只能有一个类继承Application，该项目创建的时候会自动执行Application 的onCreate()方法。所以导入项目的Application类的onCreate()方法并没有执行，在里面做的初始化操作也就没有执行。解决方法是用自己项目的Application类去继承导入项目的Application类。

7. 如果自己项目的so包放在了armeabi，armeabi-v7a和x86三个目录下，如果导入的项目也有依赖so包，那么也需要把依赖的so包分别放在这3个目录下，不然执行System.LoadLibrary()会报错。

8. 直接运行没有问题后，打的包运行就报错，可能是混淆的问题，需要将导入项目的包全部keep掉。

9. 去掉所有重复的jar包和重复的类之后，还报java.exe返回值不是0而是1,2之类的，可能是jar包方法数太多超过了65k的限制，可以在gradle文件中加入:

		android.dexOptions {
		    jumboMode = true
		    javaMaxHeapSize “2g”
		}
10. 导入的项目中如果有布局文件名称和主项目中的布局文件名称一样的话，会导致导入项目编译的R文件出错，找不到该布局文件和布局文件中所有控件的id，解决方法是修改主项目布局文件的名字。


