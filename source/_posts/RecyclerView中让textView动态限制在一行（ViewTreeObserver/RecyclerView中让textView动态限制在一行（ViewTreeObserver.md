---
title: RecyclerView中让textView动态限制在一行（ViewTreeObserver
date: 2023-02-17 15:29:15
tags: view
categories: andorid
cover: "/images/jp_fengmian.jpg"
---


---

# 前言

关键字：OnPreDrawListener、ViewTreeObserver、正确移除、RecyclerView
---


# 一、背景
做安卓的都知道的常识，一般在一个列表中的一项文字显示不下了，一般来说是限制固定行数并在其后方添加省略号，有一次做需求开开心心的写完，结果UI验收时告诉我说这个数据比较重要，需要全部展示，但是呢数据又不能超过一行，这就比较难办了，我好生说这个功能项目中没有，我也没做过，要不然给我一天我研究一下，要不然呢就缓一缓，然后UI就去问了他们管事的，UI管事的说，怎么能做不了呢，不可能需要一天，今天必须要，并反馈给了我的部长，当时我的心情是这样

dog！




# 二、解决

## 1.项目中的使用
求助同事后在项目中使用下例代码来动态改变textView行数
```java
    private void salaryTextObServer(TextView textView, float textSize) {
        textView.getViewTreeObserver().addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener() {
            @Override
            public boolean onPreDraw() {
            //监听移除，不然就会嘿嘿嘿
                textView.getViewTreeObserver().removeOnPreDrawListener(this);
                int line = textView.getLineCount();
                if (line > 1) {
                    textView.setTextSize(TypedValue.COMPLEX_UNIT_SP, textSize - 1);
                    salaryTextObServer(textView, textSize - 1);
                } else {
                }
                return false;
            }
        });
    }
```
将需要动态改变行数的textView传入就好了，挺方便并且简单。

## 2.学习并理解
平常绘制自定义view或者用画笔绘制自定义view等都没有使用到ViewTreeObserver这个类，本着学习进步的想法，在网上也搜索了相关资料，算是自问自答吧，整理出来。

### a.ViewTreeObserver是干嘛的？
顾名思义，是一个使用观察者模式的view的观测类，当我们需要动态观测View的宽高等属性时可以使用，ViewTreeObserver类下有如下方法可以使用，分别适用于特定场景，酌情使用

 - addOnWindowAttachListener 窗口附加监听器
 - addOnWindowFocusChangeListener  窗口焦点监听器
 - addOnGlobalFocusChangeListener 全局焦点监听器
 - addOnGlobalLayoutListener 全局布局监听器
 - addOnPreDrawListener 预绘制监听器（这也是本篇文章使用的，我理解为在view绘制之前）
 - addOnDrawListener 绘制监听器
 

### a.ViewTreeObserver有哪些坑？
1.以上监听器都有与之对应的remove方法，在添加监听器后都需要在方法中remove掉，不然会报错（什么你问为什么会报错？就像让你一直打望一个美女，你能坚持一整天吗？总归是要休息的嘛），并且如本文例子一样，如果在列表中使用，getViewTreeObserver()时加上具体view控件。（什么你又问为什么？就像让你从一群鸭子里面找一只丑小鸭，总归是很费劲的嘛）
2.[View#getViewTreeObserver每次得到的ViewTreeObserver可能不是同一个实例](https://juejin.cn/post/6844903858511020039)
3.[ViewTreeObserver在RecyclerView中的使用](https://juejin.cn/post/6844903939339468814)
 
---

# 总结
学习如逆水行舟，不进则退

