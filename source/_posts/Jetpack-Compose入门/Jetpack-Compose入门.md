---
title: Jetpack Compose入门
date: 2023-04-19 11:01:01
tags: Jetpack Compose
categories: andorid
thumbnail: "/images/jp_fengmian.jpg"
---

# 一.前言

本篇文章将简单介绍 **Jectpack Compose**,只带领大家认识在安卓中Compose的使用，既然要使用**Jectpack Compose**，那就必须踩一捧一了（doge）

# 二.原生View的缺点与Compose的优点

## 原生View的缺点

> 以下仅代表个人观点，不可否认view也有自己的优点

## 臃肿

所有人一入行Android都是从xml布局开始的，都是从view开始的，不得不说作为一个基类，它确实很强大，很成熟，但是相信大部分人都不没有去看看view的源码有多少行，今天告诉大家，一共有3468行。

![image-20230403175158572](/images/image-20230403175158572.png)

是的，作为一个基类，它太臃肿了。

---



## 使用麻烦

大家写xml都写习惯了，各种属性熟记于心，所以并不觉得什么，那么我们通过一个小例子来对比一下

用原生View的方式声明一个文本，不谈动态更改值，你得声明一个xml吧

```kotlin
<TextView
    android:id="@+id/text"
    android:text="这是一条文本"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>
```

而Compose结合kotlin的Lamda，声明一个文本，只需要一行代码

```kotlin
Text("这是一条文本")
```

---

这两种方式的实现效果是一样的，但是很直观的你可以看到

## 预览功能不够强大

好了有些杠精就要说了“我可以在xml布局中预览啊”，你先别急，我们再举个在工作当中的例子，当你美滋滋的完成了开发任务，正在进行UI验收时，UI跑过来跟你说，这个发布页下面有一根分割线，我的UI图里分割线是1.dp，你这里有2.dp；你看着你在xml中的预览，确实是1.dp啊，你说好我看下，一跑项目十五分钟过去了，发现确实是2.dp然后再回过头来看为什么，这但是Compose就可以直接指定运行某个页面，甚至可以切换主题和交互，而不是像xml只能看看。

**综上所诉，高下立判，xml在现在看来确实不够优雅了**

---

# Compose的优点



### 为什么采用 Compose

Jetpack Compose 是用于构建原生 Android 界面的新工具包。它使用更少的代码、强大的工具和直观的 Kotlin API，可以帮助您简化并加快 Android 界面开发，打造生动而精彩的应用。它可让您更快速、更轻松地构建 Android 界面。

## 更少的代码

编写更少的代码会影响到所有开发阶段：作为代码撰写者，需要测试和调试的代码会更少，出现 bug 的可能性也更小，您就可以专注于解决手头的问题；作为审核人员或维护人员，您需要阅读、理解、审核和维护的代码就更少。

与使用 Android View 系统（按钮、列表或动画）相比，Compose 可让您使用更少的代码实现更多的功能。无论您需要构建什么内容，现在需要编写的代码都更少了。以下是我们的一些合作伙伴的感想：

- “对于相同的 Button 类，代码的体量要小 10 倍。”(Twitter)
- “使用 RecyclerView 构建的任何屏幕（我们的大部分屏幕都使用它构建）的大小也显著减小。”(Monzo)
- ““只需要很少几行代码就可以在应用中创建列表或动画，这一点令我们非常满意。对于每项功能，我们编写的代码行更少了，这让我们能够将更多精力放在为客户提供价值上。”(Cuvva)

编写代码只需要采用 Kotlin，而不必拆分成 Kotlin 和 XML 部分：“当所有代码都使用同一种语言编写并且通常位于同一文件中（而不是在 Kotlin 和 XML 语言之间来回切换）时，跟踪变得更容易”(Monzo)

无论您要构建什么，使用 Compose 编写的代码都很简洁且易于维护。“Compose 的布局系统在概念上更简单，因此可以更轻松地推断。查看复杂组件的代码也更轻松。”(Square)

## 直观

Compose 使用声明性 API，这意味着您只需描述界面，Compose 会负责完成其余工作。这类 API 十分直观 - 易于探索和使用：“我们的主题层更加直观，也更加清晰。我们能够在单个 Kotlin 文件中完成之前需要在多个 XML 文件中完成的任务，这些 XML 文件负责通过多个分层主题叠加层定义和分配属性。”(Twitter)

利用 Compose，您可以构建不与特定 activity 或 fragment 相关联的小型无状态组件。这让您可以轻松重用和测试这些组件：“我们给自己设定的目标是，交付一组新的无状态界面组件，确保它们易于使用和维护，且可直观实现/扩展/自定义。就这一点而言，Compose 确实为我们提供了一个可靠的答案。”(Twitter)

在 Compose 中，状态是显式的，并且会传递给相应的可组合项。这样一来，状态便具有单一可信来源，因而是封装和分离的。然后，应用状态变化时，界面会自动更新。“在对某些内容进行推断时，不必处理太多信息，并且无法控制或难以理解的行为也更少”(Cuvva)

## 加速开发

Compose 与您所有的现有代码兼容：您可以从 View 调用 Compose 代码，也可以从 Compose 调用 View。大多数常用库（如 Navigation、ViewModel 和 Kotlin 协程）都适用于 Compose，因此您可以随时随地开始采用。“我们集成 Compose 的初衷是实现互操作性，我们发现这件事情已经‘水到渠成’。我们不必考虑浅色模式和深色模式等问题，整个体验无比顺畅。”(Cuvva)

借助全面的 Android Studio 支持以及实时预览等功能，您可以更快地迭代和交付代码：“*Android Studio 中的预览功能极大地节省了我们的时间。构建多个预览的能力也帮我们节省了时间。我们通常需要检查不同状态下或采用不同设置的界面组件（例如错误状态或采用不同的字体大小等）。由于能够创建多个预览，我们可以轻松执行这些检查。*”(Square)

## 功能强大

利用 Compose，您可以凭借对 Android 平台 API 的直接访问和对于 Material Design、深色主题、动画等的内置支持，创建精美的应用：“Compose 不仅解决了声明性界面的问题，还改进了无障碍功能 API、布局等各种内容。将设想变为现实所需的步骤更少了”(Square)。

利用 Compose，您可以轻松快速地通过动画让应用变得生动有趣：“在 Compose 中添加动画效果非常简单，没有理由不去为颜色/大小/高度变化添加动画效果”(Monzo)，“不需要任何特殊的工具就能制作动画，这与显示静态屏幕没有什么不同”(Square)。

无论您是使用 Material Design 还是自己的设计系统进行构建，Compose 都可以让您灵活地实现所需的设计：“从基础上将 Material Design 分离出来对我们来说非常有用，因为我们要构建自己的设计系统，这往往需要与 Material 不同的设计要求。”(Square)



# 三.Hello Compose

光说不练假把式，我们马上操练起来。

##	先new 一个 project



![image-20230404102128397](/images/image-20230404102128397.png)

## 然后选择创建一个空的Compose 项目

> 注意只有新版AS才有这个选项

![image-20230404102342123](/images/image-20230404102342123.png)

---

名字就叫Hello Compose吧

![image-20230404102915709](/images/image-20230404102915709.png)

finish

---

编译完成后我们可以看到这样一个界面

<img src="/images/image-20230404110658795.png" alt="image-20230404110658795" style="zoom: 200%;" />

可以看到一个适合Compose开发的环境已经搭建好了，后面的介绍也是基于此环境的。

---

### setContent函数

与常规的setContentView函数不同setContent函数必须是activity继承ComponentActivity才能使用，并且其接收参数为声明UI内容的@Composable函数。

### **@Composable** 注解

我们可以看到页面中的带有 **@Composable** 注解的函数 **Greeting()** 与**DefaultPreview()**,它们都是被声明为UI内容的，换句话说，它们都是页面；字面意思来说，它是一个可组合项。（ps：下文所说的可组合项都是指代带有 **@Composable** 注解的函数）

### @Preview 注解

这是一个配合**@Composable** 注解使用的注解，标识着可组合项可以被预览，这个注解的强大在于可以运行单个页面和查看其点击效果。试想一下如果我们只运行一个页面在手机上，是不是会快很多。

![在这里插入图片描述](/images/3B3A9318-411D-4B89-B6CD-7C756B346216.png)

这是最简单的使用，我们点进@Preview的源码

参数：

 - name-此预览的显示名称，允许在面板中识别它。

  - group-此@Preview的组名称。这允许在UI中对它们进行分组，并仅显示其中一个或多个。

  - apiLevel—呈现带注释的@Composable时使用的API级别

 - widthDp—将在其中渲染带注释的@Composable的最大DP宽度。使用此选项可以限制渲染视口的大小。

 - heightDp-将在其中渲染注释的@Composable的最大高度（以DP为单位）。使用此选项可以限制渲染视口的大小。

 - locale-区域设置的当前用户首选项，对应于区域设置资源限定符。默认情况下，将使用默认文件夹。

 - fontScale-用户首选字体的缩放因子，相对于基本密度缩放。

 - showSystemUi-如果为true，将显示设备的状态栏和操作栏。@Composable将在完整活动的上下文中呈现。

 - showBackground-如果为true，@Composable将使用默认背景色。

  - backgroundColor-背景的32位ARGB颜色int，如果未设置，则为0

   - uiMode—根据android.content.res.Configuration.uiMode的ui模式位掩码

  - device—指示要在预览中使用的设备的设备字符串。
    ![在这里插入图片描述](/images/361e12b1a2c64b41aa9494d52012bc15.png)

我们将DefaultPreview改一下（ps：locale为设置不同的用户语言区域的参数；参数 uiMode 可接受任意 Configuration.UI_* 常量，并允许您相应地更改预览的行为。例如，您可以将预览设置为夜间模式，看看主题如何响应。本例中未体现）

```kotlin
@Preview(name = "name",
    group = "group",
    widthDp = 100,
    heightDp = 100,
    fontScale = 2f,
    backgroundColor = 0xFF00FF00,
    showBackground = true,
    showSystemUi = true,
    device = Devices.NEXUS_5
)
@Composable
fun DefaultPreview() {
    PlayComposeTheme {
        Greeting("Android")
    }
}
```

预览效果如下

![在这里插入图片描述](/images/6b03da867db348c6a4acb57fdfdb3da7.png)
注意点
**showBackground 必须为true，backgroundColor 属性才生效，并且backgroundColor 是 ARGB Long，而不是 Color 值**
**同一个函数可以使用多个@Preview注解**

```kotlin
@Preview(
    name = "small font",
    group = "font scales",
    fontScale = 0.5f
)
@Preview(
    name = "large font",
    group = "font scales",
    fontScale = 1.5f
)
@Composable
fun DefaultPreview() {
    PlayComposeTheme {
        Greeting("Android")
    }
}
```

![在这里插入图片描述](/images/6e3ebe99526e4bcba3e09b30899629d9.png)



> 如果要传递参数，还有**@PreviewParameter注解**可以配合使用。

## 主题

![image-20230404114115004](/images/image-20230404114115004.png)

再来看看项目的结构，项目创建成功过后左侧自动帮我们建好了一个叫ui.theme的文件夹，这是Compose提供给开发者控制颜色，形状,主题，字体的文件夹，用以替代xml布局当中的drawble,values文件夹，不难看出以后Compose成熟后的想法，讲个笑话 **官网说：”我们会支持java！“**



我们主要讲下主题

```kotlin
package com.zyf.hellocompose.ui.theme

import androidx.compose.foundation.isSystemInDarkTheme
import androidx.compose.material.MaterialTheme
import androidx.compose.material.darkColors
import androidx.compose.material.lightColors
import androidx.compose.runtime.Composable

private val DarkColorPalette = darkColors(
    primary = Purple200,
    primaryVariant = Purple700,
    secondary = Teal200
)

private val LightColorPalette = lightColors(
    primary = Purple500,
    primaryVariant = Purple700,
    secondary = Teal200

    /* Other default colors to override
    background = Color.White,
    surface = Color.White,
    onPrimary = Color.White,
    onSecondary = Color.Black,
    onBackground = Color.Black,
    onSurface = Color.Black,
    */
)

@Composable
fun HelloComposeTheme(darkTheme: Boolean = isSystemInDarkTheme(), content: @Composable () -> Unit) {
    val colors = if (darkTheme) {
        DarkColorPalette
    } else {
        LightColorPalette
    }

    MaterialTheme(
        colors = colors,
        typography = Typography,
        shapes = Shapes,
        content = content
    )
}
```



- DarkColorPalette与LightColorPalette

深色主题与亮色主题，如果我们想要更多颜色的主题可以在这里定义，通过定义主题的主要颜色，主要变体及次要颜色等属性配置你想要的主题

- HelloComposeTheme

  项目中使用的主题，显而易见这是一个可组合项，通过判定系统的主题来切换合适的样式。一般在setContent{}中 包裹页面使用

  ---

  

## 布局

复习一下xml布局中常用的五大布局

- [1.线性布局](https://blog.csdn.net/weixin_45932709/article/details/123965369#1_3)

- [2.相对布局](https://blog.csdn.net/weixin_45932709/article/details/123965369#2_37)

- [3.帧布局](https://blog.csdn.net/weixin_45932709/article/details/123965369#3_111)

- [4.表格布局（不常用）](https://blog.csdn.net/weixin_45932709/article/details/123965369#4_129)

- [5.绝对布局（不常用）](https://blog.csdn.net/weixin_45932709/article/details/123965369#5_153)

  ---

在Compose中，也有几种布局概念，分别为

### Column

```kotlin
@Composable
fun Greeting(name: String) {
    Column() {
        Text(text = "Hello $name!")
        Text(text = "Hello $name!")
    }
}

@Preview(showBackground = true, showSystemUi = true)
@Composable
fun DefaultPreview() {
    HelloComposeTheme {
        Greeting("Compose")
    }
}
```

垂直排列元素的布局

![image-20230404144803400](/images/image-20230404144803400.png)

---

### Row

```kotlin
@Composable
fun Greeting(name: String) {
//    Column() {
//        Text(text = "Hello $name!")
//        Text(text = "Hello $name!")
//    }

    Row() {
        Text(text = "Hello $name!")
        Text(text = "Hello $name!")
    }
}

@Preview(showBackground = true, showSystemUi = true)
@Composable
fun DefaultPreview() {
    HelloComposeTheme {
        Greeting("Compose")
    }
}
```

水平排列元素的布局

![image-20230404145358361](/images/image-20230404145358361.png)

---

### Box

```
@Composable
fun Greeting(name: String) {
    Box() {
        Text(text = "Hello $name!",Modifier.background(Color.Green).size(200.dp))
        Text(text = "11111111111111111111",Modifier.background(Color.Yellow))
    }
}

@Preview(showBackground = true, showSystemUi = true)
@Composable
fun DefaultPreview() {
    HelloComposeTheme {
        Greeting("Compose")
    }
}
```

按照代码顺序堆砌在布局中

![image-20230404150122732](/images/image-20230404150122732.png)

上诉代码中**Modifier**为修饰符，其作用在于修饰其可组合项可以更改可组合项的大小、布局、外观，我们就更改了布局的背景和尺寸以更好的表现Box的特性

---

### ConstraintLayout

这个布局要使用需要添加单独的依赖

```
implementation "androidx.constraintlayout:constraintlayout-compose:1.0.1"
```



```kotlin
@Composable
fun Greeting(name: String) {
    ConstraintLayout(){
        val (view1, view2,view3) = createRefs()
        Box(
            Modifier.constrainAs(view1) {
                top.linkTo(parent.top)
                start.linkTo(parent.start)
                end.linkTo(parent.end)
                bottom.linkTo(parent.bottom)
            }
                .background(Color.Green)
                .size(200.dp)
        )
        Box(
            Modifier.constrainAs(view2) {
                top.linkTo(view1.bottom)
                start.linkTo(view1.start)
            }
                .background(Color.Yellow)
                .size(200.dp)
        )
        Box(
            Modifier.constrainAs(view3) {
                bottom.linkTo(view1.top)
                start.linkTo(view1.start)
            }
                .background(Color.Blue)
                .size(200.dp)
        )
    }
}

@Preview(showBackground = true, showSystemUi = true)
@Composable
fun DefaultPreview() {
    HelloComposeTheme {
        Greeting("Compose")
    }
}    
```

代码将根据约束来确定自己的位置，要使用ConstraintLayout需要经历下列几步操作

- 首先要使用**createRefs()**创建约束

- 然后通过**Modifier.constrainAs()**绑定约束和制定约束

  ![image-20230404152220446](/images/image-20230404152220446.png)

  

  

# 四.小试牛刀

相信看了上述的代码过后，你会发现其实很多东西和xml还是互通的，万变不离其宗嘛，只是换了一个表达方式而已；

最后，我们通过一个分享页面的绘制来对比一下两种方式的差别，顺便熟悉下可组合项的使用方法

  ![image-20230404152220446](/images/80558c4acda64b398e9a52843cb831a5.png)

上图是我用xml实现的，点击按钮过后弹出二维码分享页面，下面是xml代码

```kotlin
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#99000000"
        android:orientation="vertical">

        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_above="@+id/bottom_layout">

            <LinearLayout
                android:id="@+id/ll_qr_code"
                android:layout_width="@dimen/dp270"
                android:layout_height="@dimen/dp370"
                android:layout_centerInParent="true"
                android:background="@mipmap/self_qr_bg"
                android:orientation="vertical">


                <ImageView
                    android:id="@+id/iv_avatar_address"
                    android:layout_width="@dimen/dp40"
                    android:layout_height="@dimen/dp40"
                    android:layout_gravity="center"
                    android:layout_marginTop="@dimen/dp13"
                    android:src="@mipmap/ic_show_qccode_head" />

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center"
                    android:layout_marginTop="@dimen/dp4"
                    android:text="邀请你加入工队"
                    android:textColor="@color/color_1C1C1C"
                    android:textSize="@dimen/sp13" />


                <TextView
                    android:id="@+id/tv_title"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center"
                    android:layout_marginTop="@dimen/dp3"
                    android:ellipsize="end"
                    android:singleLine="true"
                    android:textColor="@color/color_969696"
                    android:textSize="@dimen/sp12" />

                <ImageView
                    android:id="@+id/iv_qr_code"
                    android:layout_width="@dimen/dp148"
                    android:layout_height="@dimen/dp148"
                    android:layout_gravity="center"
                    android:layout_marginTop="@dimen/dp45"
                    android:src="@mipmap/add" />

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center"
                    android:layout_marginTop="@dimen/dp20"
                    android:text="扫码立即加入"
                    android:textColor="@color/color_666666"
                    android:textSize="@dimen/sp12" />

            </LinearLayout>

        </RelativeLayout>

        <RelativeLayout
            android:id="@+id/bottom_layout"
            android:layout_width="match_parent"
            android:layout_height="@dimen/dp167"
            android:layout_alignParentBottom="true"
            android:background="@drawable/shape_top_round_white_14"
            android:orientation="vertical">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="@dimen/dp25"
                android:orientation="horizontal"
                android:paddingStart="@dimen/dp8"
                android:paddingEnd="@dimen/dp8">

                <TextView
                    android:id="@+id/tv_dingding"
                    android:layout_width="@dimen/dp45"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:drawableTop="@mipmap/icon_details_share_dingding"
                    android:drawablePadding="@dimen/dp8"
                    android:gravity="center_horizontal"
                    android:minWidth="@dimen/dp45"
                    android:text="钉钉"
                    android:textColor="@color/color_666666"
                    android:textSize="12sp" />

                <TextView
                    android:id="@+id/tv_qq"
                    android:layout_width="@dimen/dp45"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:drawableTop="@mipmap/icon_details_share_qq"
                    android:drawablePadding="@dimen/dp8"
                    android:gravity="center_horizontal"
                    android:text="QQ"
                    android:textColor="@color/color_666666"
                    android:textSize="12sp" />

                <TextView
                    android:id="@+id/tv_wechat"
                    android:layout_width="@dimen/dp45"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:drawableTop="@mipmap/icon_details_share_wechat"
                    android:drawablePadding="@dimen/dp8"
                    android:gravity="center_horizontal"
                    android:text="微信好友"
                    android:textColor="@color/color_666666"
                    android:textSize="12sp" />

                <TextView
                    android:id="@+id/tv_wechat_moments"
                    android:layout_width="@dimen/dp45"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:drawableTop="@mipmap/icon_details_share_wechat_moments"
                    android:drawablePadding="@dimen/dp8"
                    android:gravity="center_horizontal"
                    android:text="朋友圈"
                    android:textColor="@color/color_666666"
                    android:textSize="12sp" />

                <TextView
                    android:id="@+id/tv_copy"
                    android:layout_width="@dimen/dp45"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:drawableTop="@mipmap/icon_details_share_copy"
                    android:drawablePadding="@dimen/dp8"
                    android:gravity="center_horizontal"
                    android:text="复制链接"
                    android:textColor="@color/color_666666"
                    android:textSize="12sp" />

                <TextView
                    android:id="@+id/tv_save"
                    android:layout_width="@dimen/dp45"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:drawableTop="@mipmap/icon_details_share_save"
                    android:drawablePadding="@dimen/dp8"
                    android:gravity="center_horizontal"
                    android:text="保存图片"
                    android:textColor="@color/color_666666"
                    android:textSize="12sp" />

            </LinearLayout>

            <TextView
                android:id="@+id/tv_cancel"
                android:layout_width="match_parent"
                android:layout_height="@dimen/dp48"
                android:layout_alignParentBottom="true"
                android:background="@color/white"
                android:gravity="center"
                android:text="取消"
                android:textColor="@color/color_666666"
                android:textSize="16sp" />

        </RelativeLayout>

    </RelativeLayout>

```

因为是透明弹窗，所以需要一个透明弹窗的样式，这个compose也是一样的

```kotlin
    <!--透明activity的style-->
    <style name="transparent_activity">
        <item name="android:windowBackground">@color/transparent</item>
        <item name="android:windowNoTitle">true</item>
        <item name="android:windowIsTranslucent">true</item>
        <item name="android:windowAnimationStyle">@android:style/Animation.Translucent</item>
    </style>
```

## Compose实现

通过上面的xml布局不难看出，我们想要实现的这个分享页面，结构如下
  ![image-20230404152220446](/images/975cb32ea56f4b50aa1e262e56eab84c.png)
接下来我们将分享页面拆分为 二维码分享组件和底部分享组件来实现，底部分享组件在下方，二维码分享组件在中间，所以我采用了约束布局

```kotlin
//分享页面
@Composable
fun ShareQRCode() {
    ConstraintLayout {
        val (bottomLayout, llQrcode) = createRefs()

		//二维码分享组件
        LinearLayoutQrcode(
            Modifier.constrainAs(llQrcode){
                top.linkTo(parent.top)
                start.linkTo(parent.start)
                end.linkTo(parent.end)
                bottom.linkTo(parent.bottom)
            }
        )

		//底部分享组件
        BottomLayout(
            modifier = Modifier.constrainAs(bottomLayout) {
                bottom.linkTo(parent.bottom)
            }
            
        )
    }
}
```

## 1.底部分享组件

让我们来分析下，底部分享组件需要哪些元素
![在这里插入图片描述](/images/7e64312536814309903d71be14bfd921.png)
所以我们需要这样一个结构来实现

```kotlin
Column{//背景需要半圆角

 Row{
	//这里有六个按钮
}

//取消按钮需要在底部
Button{

}

}
```

###  a.背景半圆角

```kotlin
   Column(
        modifier = modifier
            .height(167.dp)
            .background(
                colorResource(R.color.color_f5f5f5),
                shape = RoundedCornerShape(16.dp, 16.dp, 0.dp, 0.dp)
            )
    )
```

shape 属性设置圆角，上述代码中RoundedCornerShape分别设置左上，右上，左下，右下的圆角
![在这里插入图片描述](/images/86d942f964f8449db6be0b585c29280a.png)

###  b.自定义分享按钮

 自定义分享按钮的样式是这样的
 ![在这里插入图片描述](/images/c209d631cc8449ea8e6de929929f6af6.png)
上面是一个图片，下面是文字，然后都有一个局居中，代码如下

```kotlin
@Composable
fun ImageButton(@DrawableRes id: Int, text: String, modifier: Modifier) {
    Column(
        modifier
            .width(45.dp),
    ) {
        Image(
            painter = painterResource(id = id),
            contentDescription = "",
            Modifier
                .fillMaxWidth()
                .padding(bottom = 8.dp),
            Alignment.Center
        )
        Text(
            text = text,
            fontSize = 11.sp,
            color = colorResource(id = R.color.color_666666),
            textAlign = TextAlign.Center,
            modifier = Modifier.fillMaxWidth()
        )

    }
}
```

**Alignment.Center**是一个居中属性，**TextAlign.Center**代表文字居中，但是在实际使用中，我发现单独使用Alignment.Center并不生效，于是我搭配了Modifier.fillMaxWidth()使其生效，painterResource是使用图片资源函数


然后再在row中使用，从xml中可以看到这个地方使用了权重，在compose中我们也可以用 Modifier.weight()属性来代替

```kotlin
 Row(
            Modifier
                .fillMaxWidth()
                .padding(8.dp, 25.dp, 8.dp, 0.dp)

        ) {
            ImageButton(
                id = R.mipmap.icon_details_share_dingding,
                text = "钉钉",
                Modifier.weight(1f)
            )
            ImageButton(
                id = R.mipmap.icon_details_share_qq,
                text = "QQ",
                Modifier.weight(1f)
            )
            ImageButton(
                id = R.mipmap.icon_details_share_wechat,
                text = "微信好友",
                Modifier.weight(1f)
            )
            ImageButton(
                id = R.mipmap.icon_details_share_wechat_moments,
                text = "朋友圈",
                Modifier.weight(1f)
            )
            ImageButton(
                id = R.mipmap.icon_details_share_copy,
                text = "复制链接",
                Modifier.weight(1f)
            )
            ImageButton(
                id = R.mipmap.icon_details_share_save,
                text = "保存图片",
                Modifier.weight(1f)
            )
        }
```



###   c.在底部的取消按钮

 为了学习嘛，就尝试了另外一种将控件放于底部的方式


```kotlin
 Column(
            Modifier.fillMaxHeight(),
            Arrangement.Bottom
        ){
Button（）{

	}
}
```

###   d.总体

这里为了显示点击的Toast，多传递了一个context: Activity，BottomLayout完整代码如下

```kotlin
@Composable
fun BottomLayout(
    modifier: Modifier,
    context: Activity
) {
    Column(
        modifier = modifier
            .height(167.dp)
            .background(
                colorResource(R.color.color_f5f5f5),
                shape = RoundedCornerShape(16.dp, 16.dp, 0.dp, 0.dp)
            )
    ) {
        Row(
            Modifier
                .fillMaxWidth()
                .padding(8.dp, 25.dp, 8.dp, 0.dp)

        ) {
            ImageButton(
                id = R.mipmap.icon_details_share_dingding,
                text = "钉钉",
                Modifier.weight(1f)
            )
            ImageButton(
                id = R.mipmap.icon_details_share_qq,
                text = "QQ",
                Modifier.weight(1f)
            )
            ImageButton(
                id = R.mipmap.icon_details_share_wechat,
                text = "微信好友",
                Modifier.weight(1f)
            )
            ImageButton(
                id = R.mipmap.icon_details_share_wechat_moments,
                text = "朋友圈",
                Modifier.weight(1f)
            )
            ImageButton(
                id = R.mipmap.icon_details_share_copy,
                text = "复制链接",
                Modifier.weight(1f)
            )
            ImageButton(
                id = R.mipmap.icon_details_share_save,
                text = "保存图片",
                Modifier.weight(1f)
            )
        }

        Column(
            Modifier.fillMaxHeight(),
            Arrangement.Bottom
        ) {
            Button(
                onClick = {
                    Toast.makeText(context, "取消分享", Toast.LENGTH_SHORT).show()
                    context.finish()

                },
                Modifier.fillMaxWidth(),
                colors = ButtonDefaults.buttonColors(
                    containerColor = Color.White
                ),
                shape = RoundedCornerShape(0.dp)//圆角

            ) {
                Text(
                    text = "取消",
                    fontSize = 16.sp,
                    textAlign = TextAlign.Center,
                    modifier = Modifier
                        .fillMaxWidth()
                )
            }

        }

    }
}


@Composable
fun ImageButton(@DrawableRes id: Int, text: String, modifier: Modifier) {
    Column(
        modifier
            .width(45.dp),
    ) {
        Image(
            painter = painterResource(id = id),
            contentDescription = "",
            Modifier
                .fillMaxWidth()
                .padding(bottom = 8.dp),
            Alignment.Center
        )
        Text(
            text = text,
            fontSize = 11.sp,
            color = colorResource(id = R.color.color_666666),
            textAlign = TextAlign.Center,
            modifier = Modifier.fillMaxWidth()
        )

    }
}
```

效果如下
![在这里插入图片描述](/images/7344695b99454ad5a355f893d7ffc12c.jpeg)





## 2.二维码分享组件

一样的我们先分析一下
![在这里插入图片描述](/images/c0053df1dead413982c1524b3d9992d0.png)
可以看出二维码分享组件还是比较简单，当然在绘制组件前还要定位

### a.组件居中

首先我们让它显示出来

```kotlin
@Composable
fun LinearLayoutQrcode(modifier: Modifier){
    Column(
        modifier.paint(painterResource(id = R.mipmap.self_qr_bg))
            .width(270.dp)
            .height(370.dp)
    ) {

    }
}
```

运行出来效果如下，显而易见，这和我们理想中的效果有点偏差
![在这里插入图片描述](/images/00145d66ded7497b8250a6950afb373e.png)
对比了下之前的xml布局，发现我们在compose中的组件是居于父布局居中的，xml布局是上半部分布局居中的，下面我们来修改一下，将ConstraintLayout中的居中取消，并在Column中居中

```kotlin
//引用
        LinearLayoutQrcode(
            Modifier.constrainAs(llQrcode){
//                top.linkTo(parent.top)
//                start.linkTo(parent.start)
//                end.linkTo(parent.end)
//                bottom.linkTo(parent.bottom)
            }
        )


@Composable
fun LinearLayoutQrcode(modifier: Modifier){
    Column(
        modifier.paint(painterResource(id = R.mipmap.self_qr_bg))
            .fillMaxSize(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally

    ) {

    }
}

```

然而并没有变化，给我干蒙了，在仔细检查后，我觉得可能是因为LinearLayoutQrcode使用的是父布局的modifier，所以fillMaxSize属性也是父布局的，所以我用了另外一个办法，先让LinearLayoutQrcode组件设置一个和BottomLayout高度一样的外边距，然后再让它居中，就能实现我们想要的效果，可是compose没有外边距，所以我多包了一层Column，代码如下

```kotlin
//引用
        LinearLayoutQrcode(
            Modifier.constrainAs(llQrcode){
                top.linkTo(parent.top)
                start.linkTo(parent.start)
                end.linkTo(parent.end)
                bottom.linkTo(parent.bottom)
            }
        )


@Composable
fun LinearLayoutQrcode(modifier: Modifier){
    Column(
        modifier
            .padding(bottom = 167.dp)
    ) {

        Column(
            Modifier.paint(painterResource(id = R.mipmap.self_qr_bg)),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {

        }
    }
}
```

效果
![在这里插入图片描述](/images/a72ff79407e243d0b66a5f1609d8fd35.jpeg)
可以看到，非常理想啊，我们继续下一步

### b.顶部头像

```kotlin
 Image(
                painter = painterResource(id = R.mipmap.ic_show_qccode_head),
                contentDescription = "默认头像",
                Modifier
                    .width(53.dp)
                    .height(53.dp)
                    .padding(top = 13.dp)
            )
            Text(
                text = "邀请你加入工队",
                color = colorResource(id = R.color.color_1C1C1C),
                fontSize = 13.sp,
                textAlign = TextAlign.Center,
                modifier = Modifier
                    .fillMaxWidth()
            )
            Text(
                text = "水电费-试一试1-施工队", color = colorResource(id = R.color.color_969696),
                fontSize = 12.sp,
                textAlign = TextAlign.Center,
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(top = 3.dp)
            )
```

这就没啥好说的，比较简单，有点不习惯的就是没有了外边距，好难受好难受
![在这里插入图片描述](/images/80558c4acda64b398e9a52843cb831a5.png)

### c.二维码

最后加上二维码

```kotlin
 Image(
                painter = painterResource(id = R.mipmap.qrcode),
                contentDescription = "默认头像",
                Modifier
                    .width(193.dp)
                    .height(193.dp)
                    .padding(top = 45.dp),

                )

            Text(
                text = "扫码立即加入",
                color = colorResource(id = R.color.color_666666),
                fontSize = 12.sp,
                textAlign = TextAlign.Center,
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(top = 20.dp)
            )
```

大功告成，让我们看看效果

![在这里插入图片描述](/images/80558c4acda64b398e9a52843cb831a5.png)

非常的完美啊，和xml分毫不差，大功告成，完整代码

```kotlin
package com.xf.mycomepose

import android.app.Activity
import android.content.Context
import android.os.Bundle
import android.widget.ImageButton
import android.widget.Toast
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.annotation.DrawableRes
import androidx.compose.foundation.Image
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.CornerBasedShape
import androidx.compose.foundation.shape.CornerSize
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.drawBehind
import androidx.compose.ui.draw.paint
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.graphics.RectangleShape
import androidx.compose.ui.res.colorResource
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import androidx.constraintlayout.compose.ConstraintLayout
import com.xf.mycomepose.ui.theme.MyComeposeTheme


class ShareQRCodeActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyComeposeTheme {
                // A surface container using the 'background' color from the theme
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = Color(99000000)
                ) {
                    ShareQRCode(this@ShareQRCodeActivity)
                }
            }
        }
    }

}

@Composable
fun ShareQRCode(context: Activity) {
    ConstraintLayout {
        val (bottomLayout, llQrcode) = createRefs()



        LinearLayoutQrcode(
            Modifier.constrainAs(llQrcode) {
                top.linkTo(parent.top)
                start.linkTo(parent.start)
                end.linkTo(parent.end)
                bottom.linkTo(parent.bottom)
            }
        )


        BottomLayout(
            modifier = Modifier.constrainAs(bottomLayout) {
                bottom.linkTo(parent.bottom)
            },
            context = context
        )
    }
}


@Composable
fun LinearLayoutQrcode(modifier: Modifier) {
    Column(
        modifier
            .padding(bottom = 167.dp)
    ) {

        Column(
            Modifier.paint(painterResource(id = R.mipmap.self_qr_bg)),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {

            Image(
                painter = painterResource(id = R.mipmap.ic_show_qccode_head),
                contentDescription = "默认头像",
                Modifier
                    .width(53.dp)
                    .height(53.dp)
                    .padding(top = 13.dp)
            )
            Text(
                text = "邀请你加入工队",
                color = colorResource(id = R.color.color_1C1C1C),
                fontSize = 13.sp,
                textAlign = TextAlign.Center,
                modifier = Modifier
                    .fillMaxWidth()
            )
            Text(
                text = "水电费-试一试1-施工队",
                color = colorResource(id = R.color.color_969696),
                fontSize = 12.sp,
                textAlign = TextAlign.Center,
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(top = 3.dp)
            )
            Image(
                painter = painterResource(id = R.mipmap.qrcode),
                contentDescription = "默认头像",
                Modifier
                    .width(193.dp)
                    .height(193.dp)
                    .padding(top = 45.dp),

                )

            Text(
                text = "扫码立即加入",
                color = colorResource(id = R.color.color_666666),
                fontSize = 12.sp,
                textAlign = TextAlign.Center,
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(top = 20.dp)
            )

        }
    }
}


@Composable
fun BottomLayout(
    modifier: Modifier,
    context: Activity
) {
    Column(
        modifier = modifier
            .height(167.dp)
            .background(
                colorResource(R.color.color_f5f5f5),
                shape = RoundedCornerShape(16.dp, 16.dp, 0.dp, 0.dp)
            )
    ) {
        Row(
            Modifier
                .fillMaxWidth()
                .padding(8.dp, 25.dp, 8.dp, 0.dp)

        ) {
            ImageButton(
                id = R.mipmap.icon_details_share_dingding,
                text = "钉钉",
                Modifier.weight(1f)
            )
            ImageButton(
                id = R.mipmap.icon_details_share_qq,
                text = "QQ",
                Modifier.weight(1f)
            )
            ImageButton(
                id = R.mipmap.icon_details_share_wechat,
                text = "微信好友",
                Modifier.weight(1f)
            )
            ImageButton(
                id = R.mipmap.icon_details_share_wechat_moments,
                text = "朋友圈",
                Modifier.weight(1f)
            )
            ImageButton(
                id = R.mipmap.icon_details_share_copy,
                text = "复制链接",
                Modifier.weight(1f)
            )
            ImageButton(
                id = R.mipmap.icon_details_share_save,
                text = "保存图片",
                Modifier.weight(1f)
            )
        }

        Column(
            Modifier.fillMaxHeight(),
            Arrangement.Bottom
        ) {
            Button(
                onClick = {
                    Toast.makeText(context, "取消分享", Toast.LENGTH_SHORT).show()
                    context.finish()

                },
                Modifier.fillMaxWidth(),
                colors = ButtonDefaults.buttonColors(
                    containerColor = Color.White
                ),
                shape = RoundedCornerShape(0.dp)//圆角

            ) {
                Text(
                    text = "取消",
                    fontSize = 16.sp,
                    textAlign = TextAlign.Center,
                    modifier = Modifier
                        .fillMaxWidth()
                )
            }

        }

    }
}


@Composable
fun ImageButton(@DrawableRes id: Int, text: String, modifier: Modifier) {
    Column(
        modifier
            .width(45.dp),
    ) {
        Image(
            painter = painterResource(id = id),
            contentDescription = "",
            Modifier
                .fillMaxWidth()
                .padding(bottom = 8.dp),
            Alignment.Center
        )
        Text(
            text = text,
            fontSize = 11.sp,
            color = colorResource(id = R.color.color_666666),
            textAlign = TextAlign.Center,
            modifier = Modifier.fillMaxWidth()
        )

    }
}


```

在MainActivity中跳转到这个页面

```kotlin
val intent = Intent()
        intent.setClass(this, ShareQRCodeActivity::class.java)


Button(
            onClick = {
                startActivity(mainActivity, intent, null);
            },
            modifier = Modifier.constrainAs(button) {
                top.linkTo(parent.top)
                start.linkTo(parent.start)
                end.linkTo(parent.end)
                bottom.linkTo(parent.bottom)
            }

        ) {
            Text(text = "Hello $name!")
        }

```

别忘了透明主题


