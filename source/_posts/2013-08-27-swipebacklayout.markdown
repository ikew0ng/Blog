---
layout: post
title: "SwipeBackLayout"
date: 2013-08-27 01:37
comments: true
categories: Android
---
在很久的一段时间，我都对Fuubo的滑动返回不太满意，一直想做一个更棒的滑动返回，于是便挖了这么大一个坑，意外的完成了这个滑动返回库，希望能够对各位工程师有所帮助。
欢迎各路高手加盟！

![](https://github.com/Issacw0ng/SwipeBackLayout/blob/master/art/screenshot.png?raw=true)

<!-- more -->

`SwipeBackLayout`支持左右侧滑，以及向上滑动返回，更重要的是，导入简单，直接替代原有`Activity`即可。

源代码
===
[Github](https://github.com/Issacw0ng/SwipeBackLayout)

演示Apk
===
[GooglePlay](https://play.google.com/store/apps/details?id=me.imid.swipebacklayout.demo)

使用方法
===
1. Add SwipeBackLayout as a dependency to your existing project.
2. To enable SwipeBackLayout, you can simply make your `Activity` extend `SwipeBackActivity`:
    * In `onCreate` method, `setContentView()` should be called as usual.
    * You will have access to the `getSwipeBackLayout()` method so you can customize the `SwipeBackLayout`. 
3. Make window translucent by adding `<item name="android:windowIsTranslucent">true</item>` to your theme.

示例代码
===
``` java
public class DemoActivity extends SwipeBackActivity implements View.OnClickListener {
    private int[] mBgColors;

    private static int mBgIndex = 0;

    private String mKeyTrackingMode;

    private RadioGroup mTrackingModeGroup;

    private SwipeBackLayout mSwipeBackLayout;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_demo);
        changeActionBarColor();
        findViews();
        mKeyTrackingMode = getString(R.string.key_tracking_mode);
        mSwipeBackLayout = getSwipeBackLayout();

        mTrackingModeGroup.setOnCheckedChangeListener(new RadioGroup.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(RadioGroup group, int checkedId) {
                int edgeFlag;
                switch (checkedId) {
                    case R.id.mode_left:
                        edgeFlag = SwipeBackLayout.EDGE_LEFT;
                        break;
                    case R.id.mode_right:
                        edgeFlag = SwipeBackLayout.EDGE_RIGHT;
                        break;
                    case R.id.mode_bottom:
                        edgeFlag = SwipeBackLayout.EDGE_BOTTOM;
                        break;
                    default:
                        edgeFlag = SwipeBackLayout.EDGE_ALL;
                }
                mSwipeBackLayout.setEdgeTrackingEnabled(edgeFlag);
                saveTrackingMode(edgeFlag);
            }
        });
    }
...
```
