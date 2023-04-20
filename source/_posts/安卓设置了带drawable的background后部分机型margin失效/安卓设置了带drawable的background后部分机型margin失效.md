---
title: 安卓设置了带drawable的background后部分机型margin失效
date: 2023-02-13 14:42:16
tags: 开发日常
categories: andorid
cover: "/images/andorid1.jpg"
---

# 项目背景
接收了“前辈”的宝贵布局，在此基础上更改的需求，UI验收时发现了这部分异常，真是谢谢前辈

# 问题描述
android:layout_marginLeft在设置drawable的android:background时，在部分机型上失效

```kotlin
           <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="@dimen/dp100"
                android:layout_marginLeft="@dimen/dp15"
                android:layout_marginTop="@dimen/dp10"
                android:layout_marginEnd="@dimen/dp15"
                android:background="@drawable/bg_corner5_fafafa"
                android:orientation="vertical">
```

---

# 原因分析：

但是布局预览上和部分机型是生效的，并且失效的机型在更改drawable为color时是生效的，这一度让我认为这是drawable资源文件的问题，但是我看了半天drawable资源文件也没看出来，就那么两行代码；最后发现**android:layout_marginLeft**与其他的**android:layout_marginEnd**等属性不匹配！！！！！这时我反应过来，应该是同事写代码不规范导致的

---

# 解决方案：
将**layout_marginLeft**更换为**layout_marginStart**
```kotlin
       <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginStart="@dimen/dp15"
            android:layout_marginEnd="@dimen/dp15"
            android:layout_marginTop="@dimen/dp10"
            android:layout_marginBottom="@dimen/dp15"
            android:background="@drawable/bg_corner5_fafafa"
            android:orientation="vertical">
```
代码生效，所以平时写代码还是要规范