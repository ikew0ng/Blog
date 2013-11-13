---
layout: post
title: "Android Tips: 获取资源"
date: 2013-11-13 15:39
comments: true
categories: Android
---
{% img /media/2013-11-13-get-access-to-special-res/nexus5_1.jpg 800 531 %}


个人觉得Android的资源系统比较混乱，操作资源时问Google是常有的，以下是一些常见的Case，希望给大家带来帮助。

* 获取StatusBar高度

``` java
    public static float getStatusBarHeight(Activity activity) {
        Resources resources = activity.getResources();
        int status_bar_height_id = resources.getIdentifier("status_bar_height", "dimen", "android");
        return resources.getDimension(status_bar_height_id);
    }
```

* 获取`action_bar_container`,`split_action_bar`

``` java 
public class ActionBarUtils {
    public static View findActionBarContainer(Activity activity) {
        int id = activity.getResources().getIdentifier("action_bar_container", "id", "android");
        return activity.findViewById(id);
    }

    public static View findSplitActionBar(Activity activity) {
        int id = activity.getResources().getIdentifier("split_action_bar", "id", "android");
        return activity.findViewById(id);
    }
}
```
以上方法可以用于实现点击`ActionBar`返回顶部。

* 从`Theme`中获取资源

``` java
    TypedValue value = new TypedValue();
    getTheme().resolveAttribute(R.attr.viewpager_margin, value, true);
    int color = value.data;
```
上面的代码中，从`Theme`中获取了`viewpager_margin`指定的color。





>以前幻想着离开学校之后能有更多的时间做自己喜欢的事情，现在看来，不太现实，毕竟工作
>是需要承受一定的精神压力，每晚10：30到家精神已是疲惫不堪。
>当然我也很积极的调整节奏，希望完成每天给自己定下的List，目前看来还是有成效的 一...一


题图: Nexus 5
