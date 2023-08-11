---
title: 安卓webview通用配置✈️
date: 2021-11-12 14:48:00
tags: 
    开发日常
    andorid
categories: andorid
cover: "/images/page2.webp"
---


# 主要是java的实现，解决一些白屏或者加载不出来的配置，亲测可用✔️✔️✔️

```java
webView = new WebView(mContext);
        webViewContainer.addView(webView, new FrameLayout.LayoutParams(
                FrameLayout.LayoutParams.MATCH_PARENT,
                FrameLayout.LayoutParams.MATCH_PARENT));
        webView.loadUrl(url);
        //重新加载 点击网页里面的链接还是在当前的webview里跳转。不跳到浏览器那边
        webView.setWebViewClient(new WebViewClient() {
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                return false;
            }
            // 重写此方法能够让webview处理https请求
            @Override
            public void onReceivedSslError(WebView view, SslErrorHandler handler, android.net.http.SslError error) {
                handler.proceed();
            }

            @Override
            public void onReceivedError(WebView view, int errorCode, String description, String failingUrl) {
            }
        });
        //支持js
        webView.getSettings().setJavaScriptEnabled(true);
        // 解决图片不显示
        webView.getSettings().setBlockNetworkImage(false);
        if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            webView.getSettings().setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
        }
        //自适应屏幕
        webView.getSettings().setLayoutAlgorithm(WebSettings.LayoutAlgorithm.SINGLE_COLUMN);
        webView.getSettings().setLoadWithOverviewMode(true);
        //设置可以支持缩放
        webView.getSettings().setSupportZoom(false);
        //扩大比例的缩放
        webView.getSettings().setUseWideViewPort(false);
        //设置是否出现缩放工具
        webView.getSettings().setBuiltInZoomControls(false);
		//解决白屏问题，原因不明
        webView.getSettings().setDomStorageEnabled(true);
 ```

代码的作用注释写的很清楚了，再罗列一下🔔

 - **setWebViewClient**：新加载 点击网页里面的链接还是在当前的webview里跳转。不跳到浏览器那边
 - **onReceivedSslError**：  重写此方法能够让webview处理https请求
 - **setJavaScriptEnabled**：支持js
 - **setBlockNetworkImage**，**setMixedContentMode**：解决图片不显示
 - **setLayoutAlgorithm**：自适应屏幕
 - **setSupportZoom**：设置可以支持缩放
 - **setUseWideViewPort**：扩大比例的缩放
 - **setBuiltInZoomControls**：设置是否出现缩放
 - **setDomStorageEnabled**：解决白屏问题，原因不明

# 总结
适合自己的才是最好的，不一定要加上所有配置💘
