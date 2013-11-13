---
layout: post
title: "Support for .so libraries in Gradle builds"
date: 2013-10-22 10:10
comments: true
categories: Android
---
截止目前为止`Gradle`编译还是不支持NDK的，如果要引用`.so`库，需要通过一些特殊的手段。

昨晚Google了一下，整理出如下的方法，在`Gradle-1.8`,`Android plugin for Gradle 0.6.+`的环境下测试有效：

``` java
android {
    task copyNativeLibs(type: Copy) {
        from(new File(project(':Fuubo').getProjectDir(), 'libs')) { include '**/*.so' }
        into new File(buildDir, 'native-libs')
    }

    tasks.withType(Compile) {
        compileTask -> compileTask.dependsOn copyNativeLibs
    }

    clean.dependsOn 'cleanCopyNativeLibs'

    tasks.withType(com.android.build.gradle.tasks.PackageApplication) {
        pkgTask -> pkgTask.jniDir new File(buildDir, 'native-libs')
    }
}
```

简单说来就是在打包的过程中将`libs`目录下的`.so`文件复制到`build`文件夹中，并打包到apk。


BTW，近期升级了`Android Studio 0.3`，修正了gradle导入的一些问题，以及对各种智能提示做了优化，推荐升级。