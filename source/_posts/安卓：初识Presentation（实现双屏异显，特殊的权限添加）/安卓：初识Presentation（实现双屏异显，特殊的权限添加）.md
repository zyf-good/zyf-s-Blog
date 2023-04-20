---
title: 安卓：初识Presentation（实现双屏异显，特殊的权限添加）
date:  2023-03-09 15:20:41
tags:  Presentation
categories: andorid
cover: "/images/jp_fengmian.jpg"
---
 @[TOC](安卓实现简单双屏异显)
 ### 参考资料
  链接: [Android实现双屏异显](https://blog.csdn.net/wlwl0071986/article/details/48542923?spm=1001.2101.3001.6650.15&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-15.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-15.no_search_link).
  链接: [Android 6.0:Unable to add window android.view.ViewRootImpl$W@5e2d85a -- permission denied](https://blog.csdn.net/pugongying1988/article/details/70597492).
     公司项目后期需要设备支持双屏异显，家境贫寒的我表示压根没听过，赶紧码起来，自己建了一个小demo。
# 初识Presentation
     
  我发现了Presentation，安卓官方对于Presentation是这样描述的 
![在这里插入图片描述](/images/9276ac5819cf44d092f303408f6fae2c.png)
大致意思就是，是一种特殊的对话框，是为了在辅助显示器上演示内容。

# 快速上手
想要使用Presentation，首先得找到自己的辅助屏幕，这里我们使用了安卓官方推荐的第二种方式
    创建一个activity，然后在里面写一个内部类继承于Presentation，当然我只是图方便，实际上肯定是要新建一个的
```javascript
    private class DifferentDislay extends Presentation {
        public DifferentDislay(Context outerContext, Display display) {
            super(outerContext, display);
        }

        public DifferentDislay(Context outerContext, Display display, int theme) {
            super(outerContext, display, theme);
        }

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            //这个地方的布局就是你副屏的布局拉！布局我就不贴了
            setContentView(R.layout.diffrentdisplay);
        }
    }
```
然后开始简单快乐的找副屏，找到过后显示出来
```javascript
public class MainActivity extends AppCompatActivity {

    DisplayManager mDisplayManager;//屏幕管理类

    Display[] displays;//屏幕数组

    public static int OVERLAY_PERMISSION_REQ_CODE = 99;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initPresentation();
    }

    private void initPresentation() {
        mDisplayManager = (DisplayManager) this.getSystemService(Context.DISPLAY_SERVICE);

        displays = mDisplayManager.getDisplays();

        DifferentDislay mPresentation = new DifferentDislay(this
                , displays[1]);//displays[1]是副屏

        mPresentation.getWindow().setType(

                WindowManager.LayoutParams.TYPE_SYSTEM_ALERT);

        mPresentation.show();
    }
```

然后就开始血泪填坑史，运行过后果断的闪退了，一看报错，哦，没有加权限嘛，小事，在AndroidManifest.xml中加上权限
```javascript
    <!-- 显示系统窗口权限 -->
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
```
ok，再次运行，果断又闪退，等等，我的设备是安卓7.0的，要动态申请，那就申请一个呗。可是我发现我的动态申请权限代码不起作用，最后发现这个权限要跳到单独的页面申请，完全代码如下：

# 懒人直达
```javascript
public class MainActivity extends AppCompatActivity {

    DisplayManager mDisplayManager;//屏幕管理类

    Display[] displays;//屏幕数组

    public static int OVERLAY_PERMISSION_REQ_CODE = 99;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        requestDrawOverLays();
    }

    private void initPresentation() {
        mDisplayManager = (DisplayManager) this.getSystemService(Context.DISPLAY_SERVICE);

        displays = mDisplayManager.getDisplays();

        DifferentDislay mPresentation = new DifferentDislay(this
                , displays[1]);//displays[1]是副屏  如果只有一个屏幕使用displays[0]

        mPresentation.getWindow().setType(

                WindowManager.LayoutParams.TYPE_SYSTEM_ALERT);

        mPresentation.show();
    }


    private class DifferentDislay extends Presentation {
        public DifferentDislay(Context outerContext, Display display) {
            super(outerContext, display);
        }

        public DifferentDislay(Context outerContext, Display display, int theme) {
            super(outerContext, display, theme);
        }

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.diffrentdisplay);
        }
    }


    @TargetApi(Build.VERSION_CODES.M)
    public void requestDrawOverLays() {
        if (!Settings.canDrawOverlays(MainActivity.this)) {
            Toast.makeText(this, "您还没有打开悬浮窗权限", Toast.LENGTH_SHORT).show();
            //跳转到相应软件的设置页面
            Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, Uri.parse("package:" + MainActivity.this.getPackageName()));
            startActivityForResult(intent, OVERLAY_PERMISSION_REQ_CODE);
        } else {
            // 授权成功之后执行的方法
            initPresentation();
        }
    }


    @TargetApi(Build.VERSION_CODES.M)
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == OVERLAY_PERMISSION_REQ_CODE) {
            if (!Settings.canDrawOverlays(this)) {
                Toast.makeText(this, "授权失败", Toast.LENGTH_SHORT).show();
            } else {
                Toast.makeText(this, "授权成功", Toast.LENGTH_SHORT).show();
                initPresentation();
            }
        }
    }


}
```
这样，一个简单的双屏异显demo就完成啦，共勉，别忘记加权限哦！


# 补充（TYPE_APPLICATION_OVERLAY）
后来的后来，我有一个同事要做类似的功能，我想起来有记录这部分的知识，就将这篇文章分享给她了，她使用了我的源码但是还是报了类似的没有权限的问题，这让我十分的不解,后面经过她的一番努力，发现8.0以上版本的安卓机需要将懒人直达代码中的

```kotlin
        mPresentation.getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_ALERT);
```
替换为

```kotlin
        mPresentation.getWindow().setType(WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY);
```

所以我们最终将代码更改为如下

```kotlin
 if (Build.VERSION.SDK_INT >= 26) {
                getWindow().setType(WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY);
            } else {
                getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_ALERT);
            }
```
算是一个版本适配问题吧