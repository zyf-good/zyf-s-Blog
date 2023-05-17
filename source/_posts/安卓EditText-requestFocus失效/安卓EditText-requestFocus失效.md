---
title: 安卓EditText.requestFocus失效
date: 2021-10-20 11:08:54
tags: 
    开发日常
    java
categories: andorid
cover: "/images/andorid.jpeg"
---

   之前项目需求要求在进入一个activity的时候自动获取页面内某个EditText的焦点，让用户可以进入页面过后马上输入，这里我使用了
```java
editText.requestFocus();
```
  我经过调试过后这行代码并没有生效，于是在网上查找了一些资料发现了原因，
因为我是在onCreate（）方法中调用的这行代码，但是在activity的生命周期中这个时候editText这个时候并没有被渲染在界面上，所以这行代码理所当然的失效了。

话不多说，下面上解决办法

```java
        //第一次设置延时，等待页面渲染完毕再设置焦点
        editText.postDelayed(new Runnable() {
            @Override
            public void run() {
                editText.requestFocus();
            }
        }, 500);
```
