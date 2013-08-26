---
layout: post
title: "流氓软件微信电话本案例研究"
date: 2013-08-16 23:59
comments: true
categories: Android
---
![](/media/2013-08-16-weixin-phonebook-case-study/icon.png)

前天公司的同事发现了这款流氓应用。当分析出原理时，小伙伴们都震惊了。


事情是这样的，微信电话本是最近在内测的一款通讯录应用（据说是原QQ通讯录），它最厉害的一点就是可以接管系统的联系人应用。 当你点击系统联系人，短信或者拨号（以下简称`CSP`）时，将会启动微信通讯录，而不是原有的`CSP应用`。

本文会详细解析这种偷梁换柱的拙劣手段。
<!-- more -->

相关资源
===
* [微信通讯录](http://pan.baidu.com/share/link?shareid=868146822&uk=3493082605)
* [JD-GUI](http://java.decompiler.free.fr/?q=jdgui)
* [Apktool](https://code.google.com/p/android-apktool/)


反编译APK
===
###Apktool获取资源文件
``` bash
apktool d phonebook_218_4.apk
```

###dex2jar反编译
``` bash
#重命名
mv phonebook_218_4.apk phonebook_218_4.zip
#unzip
unzip phonebook_218_4.zip
#dex2jar
dex2jar.sh classes.dex
```
最终获得的`classes_dex2jar.jar`文件需要使用`JD-GUI`查看。


源码分析
===
1. 在应用中可以找到与绑定CSP相关的设置页面

![](/media/2013-08-16-weixin-phonebook-case-study/screenshot3.png)

通过“绑定系统拨号”这个string可以找到资源文件。
``` bash
grep "绑定系统拨号" . -nrs
#找到该string的资源setting_bind_system_callbt

grep "setting_bind_system_callbt" . -nrs
#接着追setting_bind_system_callbt，可以看到这个string被某个控件引用了

grep "bind_system_callbt_switch" . -nrs
#然后通过控件resId，找到id值

grep "0x7f0c01fc" . -nrs
#就这样顺藤摸瓜追查到了com/tencent/pb/setting/controller/SettingBindActivity
```
![](/media/2013-08-16-weixin-phonebook-case-study/screenshot1.png)


2. 使用`JD-GUI`查看`SettingBindActivity.java`

![](/media/2013-08-16-weixin-phonebook-case-study/screenshot2.png)

不难看出点击button时会调用
``` java
public void a()
  {
    this.c.toggle();
    io.a().g().b("BIND_SYSTEM_SMS", this.c.isChecked());
    f();
  }
```


`io`的方法a() 是一个Singleton 暂且忽略
``` java
 public static io a()
  {
    if (a == null);
    try
    {
      if (a == null)
        a = new io();
      return a;
    }
    finally
    {
    }
  } 
```


然后注意到所有的回掉最终都会调用f方法，f方法发出了一个 action为 "bind_system_icon_intent" 的广播。
``` java
 private void f()
  {
    sendBroadcast(new Intent("bind_system_icon_intent"));
  }

  public void a()
  {
    this.c.toggle();
    io.a().g().b("BIND_SYSTEM_SMS", this.c.isChecked());
    f();
  }
```


在AndroidManifest中不难找到其对应的Receiver
```
<receiver android:name=".remote.ScreenStateReceiver">
    <intent-filter>
        <action android:name="bind_system_icon_intent" />
        <action android:name="unbind_system_icon_intent" />
    </intent-filter>
</receiver>
```


回头能追到这样一段代码
``` java 
while (!"bind_system_icon_intent".equals(paramIntent.getAction()));
    if ((bool1) || (bool2) || (bool3))
    {
      ail.a(paramContext, bool1, bool2, bool3);
      return;
    }
    ail.a();
```
显然绑定的逻辑应该在`ail`中，从`ail`参数和构造函数就可以看到一些有意思的东西
``` java 
public class ail extends View
{
  private static ail a = null;
  private static WindowManager b = null;
  private static ActivityManager c = null;
  private static aim d = null;
  private static Context e;
  private HashSet f = new HashSet();
  private HashSet g = new HashSet();
  private HashSet h = new HashSet();
  private HashSet i = new HashSet();
  private boolean j = true;
  private boolean k = true;
  private boolean l = true;
  private boolean m = false;

  private ail(Context paramContext)
  {
    super(paramContext);
    b = (WindowManager)paramContext.getSystemService("window");
    c = (ActivityManager)paramContext.getSystemService("activity");
    e = paramContext;
    e();
    ScreenStateReceiver localScreenStateReceiver = new ScreenStateReceiver();
    localScreenStateReceiver.getClass();
    d = new aim(localScreenStateReceiver, " ");
    d.start();
  }
...
}
```


`ail`继承自`View`，其中有几个值得关注的几个成员，`WindowManager b`,`ActivityManager c`,`aim d`。
分析`ail.a()`方法，核心代码创建了一个 透明窗口，`type = 2010` 查看`Android`源码2010是`TYPE_SYSTEM_ERROR`（internal system error windows, appear on top of everything they can.），也就是说这个窗口会覆盖在所有用户窗口之上。
``` java
  public static void a(Context paramContext, boolean paramBoolean1, boolean paramBoolean2, boolean paramBoolean3)
  {
    if (a == null)
      a = new ail(paramContext);
    if (a != null)
    {
      a.k = paramBoolean1;
      a.j = paramBoolean2;
      a.l = paramBoolean3;
    }
    if ((a == null) || (a.m));
    try
    {
      b.removeView(a);
      a.m = false;
      localail = a;
      localail.setBackgroundColor(0);
      localLayoutParams = new WindowManager.LayoutParams();
      localLayoutParams.width = 1;
      localLayoutParams.height = 1;
      localLayoutParams.gravity = 51;
      localLayoutParams.x = 0;
      localLayoutParams.y = 0;
      localLayoutParams.format = 1;
      localLayoutParams.type = 2010;
      localLayoutParams.flags = 262152;
      if (a.m);
    }
    catch (Exception localException2)
    {
      try
      {
        ail localail;
        WindowManager.LayoutParams localLayoutParams;
        b.addView(localail, localLayoutParams);
        a.m = true;
        return;
        localException2 = localException2;
        Object[] arrayOfObject2 = new Object[2];
        arrayOfObject2[0] = "removeView";
        arrayOfObject2[1] = localException2.getMessage();
        Log.w("gray", arrayOfObject2);
      }
      catch (Exception localException1)
      {
        Object[] arrayOfObject1 = new Object[2];
        arrayOfObject1[0] = "removeView";
        arrayOfObject1[1] = localException1.getMessage();
        Log.w("gray", arrayOfObject1);
      }
    }
  }
```


同时注意到`ail`重写了`onTouchEvent`，在touch事件中会调用`d.a()`方法。结合上面分析的悬浮窗，大概能推测，`微信通讯录`启动后会创建一个位于顶层的透明窗口，同时监听所有TouchEvent，检查用户可能启动的任何应用，如果发现是CSP就会替换为微信通讯录的`Activity`。
``` java
  public boolean onTouchEvent(MotionEvent paramMotionEvent)
  {
    d.a();
    return super.onTouchEvent(paramMotionEvent);
  }
```


接下来，就看看`微信通讯录`是如何对用户启动的应用做判断的。
首先，在`ail`构造函数中，调用了e()方法，与之相关的是方法a()，这两方法完成了对系统中所有拨号，联系人相关应用包名的搜索，并将结果保存在对应的`HashMap`中。

``` java 
private void a(Intent paramIntent, HashSet paramHashSet, boolean paramBoolean)
  {
    Iterator localIterator = e.getPackageManager().queryIntentActivities(paramIntent, 65536).iterator();
    while (true)
    {
      ResolveInfo localResolveInfo;
      if (localIterator.hasNext())
      {
        localResolveInfo = (ResolveInfo)localIterator.next();
        if (!paramBoolean)
        {
          if ((0x1 & localResolveInfo.activityInfo.applicationInfo.flags) == 0)
            continue;
          paramHashSet.add(localResolveInfo.activityInfo.name);
          if ((localResolveInfo.activityInfo.targetActivity != null) && (localResolveInfo.activityInfo.targetActivity.length() > 1))
            paramHashSet.add(localResolveInfo.activityInfo.targetActivity);
        }
      }
      else
      {
        return;
        paramHashSet.add(localResolveInfo.activityInfo.name);
      }
    }
  }


  private void e()
  {
    Intent localIntent1 = new Intent();
    localIntent1.setAction("android.intent.action.CALL_BUTTON");
    a(localIntent1, this.g, false);
    Intent localIntent2 = new Intent();
    localIntent2.setAction("android.intent.action.DIAL");
    a(localIntent2, this.g, false);
    if (this.g.size() < 1)
    {
      this.g.add("com.android.contacts.activities.DialtactsActivity");
      this.g.add("com.android.contacts.DialtactsActivity");
    }
    Intent localIntent3 = new Intent();
    localIntent3.setAction("android.intent.action.VIEW");
    localIntent3.setType("vnd.android.cursor.dir/contact");
    a(localIntent3, this.f, false);
    if (this.f.size() < 1)
      this.f.add("com.sonyericsson.android.socialphonebook");
    if (this.i.size() < 1)
    {
      this.i.add("com.sonyericsson.conversations");
      this.i.add("com.android.mms");
      this.i.add("com.motorola.blur.conversations");
    }
    Intent localIntent4 = new Intent();
    localIntent4.setAction("android.intent.action.MAIN");
    localIntent4.addCategory("android.intent.category.HOME");
    a(localIntent4, this.h, true);
    if ((IssueSettings.bO) || (IssueSettings.bP))
    {
      this.h.add("com.android.mms.ui.SingleRecipientConversationActivity");
      this.h.add("com.android.mms.ui.NewMessagePopupActivity");
    }
  }
```

在用户产生触摸事件时，调用方法 aim.a()。
``` java
  public boolean a()
  {
    this.b.removeMessages(2013);
    ail localail = ail.b();
    List localList = ail.c().getRunningTasks(1);
    if ((localList != null) && (localList.size() > 0))
    {
      String str = ((ActivityManager.RunningTaskInfo)localList.get(0)).topActivity.getClassName();
      if (!ail.g(localail).contains(str))
        return false;
      if (a(str))
      {
        this.b.sendEmptyMessageDelayed(2013, 150L);
        this.b.sendEmptyMessageDelayed(2013, 200L);
        this.b.sendEmptyMessageDelayed(2013, 300L);
        this.b.sendEmptyMessageDelayed(2013, 500L);
        return true;
      }
    }
    this.b.sendEmptyMessageDelayed(2013, 50L);
    this.b.sendEmptyMessageDelayed(2013, 100L);
    this.b.sendEmptyMessageDelayed(2013, 200L);
    this.b.sendEmptyMessageDelayed(2013, 500L);
    return true;
  }
```
看到这一段时，我有理由怀疑该程序员已经进入丧心病狂模式，每当检查到栈顶的`Activity`是CSP相关应用，就发出四条延时消息，我估计这个延时是为了确保该`Activiy`完全启动，然后将其替换。

以上分析大部分源自汪文俊大哥，非常感谢。

如何屏蔽同类应用
===
从原理出发屏蔽应用悬浮窗，目前只有MIUI和Android 4.3具备该功能。

![](/media/2013-08-16-weixin-phonebook-case-study/screenshot4.png)



结语
===
仅从技术角度来说，这段代码还是挺有趣的，但作为一个大厂的应用，随意篡改系统应用，甚至是监听用户行为，这样的做法我认为是欠考虑的。不难预见，微信通讯录仅仅是这场灾难的起始，或许未来的某一天点击微信启动QQ会是一件喜闻乐见的事情。

