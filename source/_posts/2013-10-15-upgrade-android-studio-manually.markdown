---
layout: post
title: "Upgrade Android Studio Manually"
date: 2013-10-15 23:26
comments: true
categories: 
---
升级Android Studio是一件费劲的事情，官网的链接迟迟不更新，Check update 始终Fail，好在有高手研究出了这套手动升级的方案。


1. 检查当前版本号
在`About Android Studio`选项中可以查看当前的版本号，比如升级前我的版本号为：`130.737825`


2. 查询当前最新的版本号
访问[https://dl.google.com/android/studio/patches/updates.xml](https://dl.google.com/android/studio/patches/updates.xml) 
可查询当前最新的build版本号
{% img /media/2013-10-15-upgrade-android-studio-manually/screenshot.png 800 531 %}

3. 下载增量更新包
通过上一步查询到最新的版本号为 `132.863010`，于是我们需要下载
[https://dl.google.com/android/studio/patches/AI-130.737825-132.863010-patch-mac.jar](https://dl.google.com/android/studio/patches/AI-130.737825-132.863010-patch-mac.jar)
这个patch

4. 安装更新包
``` bash
java -classpath AI-130.737825-132.863010-patch-mac.jar com.intellij.updater.Runner install /Applications/Android\ Studio.app
```
最末尾的参数是Android Studio 安装路径

Windows和Linux操作系统升级流程类似。

--- Edit ---

Canary下载 [http://tools.android.com/download/studio](http://tools.android.com/download/studio)