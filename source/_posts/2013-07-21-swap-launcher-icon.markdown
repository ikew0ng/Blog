---
layout: post
title: "动态替换桌面图标"
date: 2013-07-21 14:50
comments: true
categories: Android
---
声明：这个方法最初从SuperUser的作者博客上学习的，暂时没找到原文

替换图标的原理是动态的设置应用的`activity-alias`:

1. 在`AndroidManifest.xml`中为应用注册多个`activity-alias`，并提供不同的图标，例如在Fuubo中，我需要对mx2大图标做适配，那么第二个`activity-alias`需要使用大图标的资源文件`android:icon=”@drawable/ic_launcher_mx2″`添加权限`android.permission.KILL_BACKGROUND_PROCESSES`

2. 在运行时根据需求启动对应的`activity-alias` 并disable其他`activity-alias`

3. 重新启动launcher进程

<!--more-->


{% gist 5541030 AndroidManifest.xml %}

{% gist 5541030 SetIcon.java %}


