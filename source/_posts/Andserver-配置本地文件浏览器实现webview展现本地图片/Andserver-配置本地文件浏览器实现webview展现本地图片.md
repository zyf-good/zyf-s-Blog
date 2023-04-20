---
title: Andserver 配置本地文件浏览器实现webview展现本地图片
date: 2022-05-11 11:35:06 
tags: Andserver
categories: andorid
cover: "/images/jp_fengmian.jpg"
---



---

# 前言

接到一个需求，需要在安卓的vue的本地页面上加载本地的图片，而不是网络图片，这样当设备没有网络的时候图片也能正常显示，因为项目使用的是andserver+webview这种项目结构，用Andserver来解决这个问题最好不过；大概的思路就是先将需要加载的网络图片下载到本地，然后通过andserver提供的WebConfig增加一个类似本地的文件浏览器静态网站，当你的本地端口访问该路径时就可以加载出保存好的本地图片；

---


# 一、将图片下载到本地
封装一个写好了的下载网络图片的工具类
```java
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.os.AsyncTask;


import com.aw.ccpos.client.Client;
import com.aw.ccpos.client.logger.Logger;

import java.io.FileOutputStream;
import java.io.InputStream;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.Executor;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

import javax.net.ssl.HttpsURLConnection;

/**
 * @ProjectName : xxxx
 * @Author : yifeng_zeng
 * @Time : 2022/2/18 14:50
 * @Description : 保存网络图片到安卓本地工具类
 */
public class NetworkImageUtil {
    private static final String TAG = "NetworkImageUtil";
    private Bitmap bitmap;
    private int corePoolSize = 60;
    private int maximumPoolSize = 80;
    private int keepAliveTime = 10;
    private BlockingQueue<Runnable> workQueue = new LinkedBlockingQueue<Runnable>(maximumPoolSize);
    private Executor threadPoolExecutor = new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, TimeUnit.SECONDS, workQueue);
    public static NetworkImageUtil instance;

    public static NetworkImageUtil getInstance(){
        if(instance == null){
            instance = new NetworkImageUtil();
        }
        return instance;
    }
    /*
    * params "/fs/img/2494831762213019.猴头菇.jpeg"
    * */
    public void saveImage(String... params){
        Task task = new Task();
        task.executeOnExecutor(threadPoolExecutor,params);
    }


    /**
     * 获取网络图片
     * @param imageurl 图片网络地址
     * @return Bitmap 返回位图
     */
    public Bitmap getImageInputStream(String imageurl){
        HttpsURLConnection connection = null;
        Bitmap bitmap=null;
        try {
            URL url = new URL(imageurl);
            Logger.d(TAG, "download image with url: " +  url);
            connection = (HttpsURLConnection) url.openConnection();
            connection.setSSLSocketFactory(HttpsUtils.getSslSocketFactory(null,null,null));
            connection.connect();
            // expect HTTP 200 OK, so we don't mistakenly save error
            // report
            // instead of the file
            if (connection.getResponseCode() != HttpURLConnection.HTTP_OK) {
                Logger.e(TAG, "Server returned HTTP "+connection.getResponseCode() + " "
                        + connection.getResponseMessage() );
            }
            InputStream inputStream=connection.getInputStream();
            bitmap= BitmapFactory.decodeStream(inputStream);
            inputStream.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return bitmap;
    }
    /**
     * 保存位图到本地
     * @param bitmap
     * @param path 本地路径
     * @return void
     */
    public void saveImage(Bitmap bitmap, String path){
//        File file=new File(path);
        FileOutputStream fileOutputStream=null;
        //文件夹不存在，则创建它
//        if (!file.exists()) {
//            file.mkdirs();
//        }
        try {
            fileOutputStream=new FileOutputStream("/sdcard/image"+path);
            bitmap.compress(Bitmap.CompressFormat.JPEG, 100,fileOutputStream);
            fileOutputStream.close();
            Logger.d(TAG, "SavaImage: success: "+path);
        } catch (Exception e) {
            Logger.e(TAG, "SavaImage: fail: "+path);
            e.printStackTrace();
        }
    }

    /**
     * 异步线程下载图片
     *
     */
    class Task extends AsyncTask<String, Integer, String> {

        @Override
        protected String doInBackground(String... params) {
            if (bitmap!=null){
                bitmap=null;
            }
            bitmap=NetworkImageUtil.getInstance().getImageInputStream((String)params[0]);
            NetworkImageUtil.getInstance().saveImage(bitmap, (String)params[0]);
            return null;
        }

        @Override
        protected void onPostExecute(String result) {

        }

    }


}

```
注释写的挺详细了，这里就不多赘述，简单提下HttpsUtils是封装的一个HttpsUtils的工具类，有需要可以去我的这篇文章里去找，这也是一篇使用andserver的文章，侧重点不一样，如果你的网络图片是http的，当我没说。
 链接:文章地址 [andriod andserver 全局转发代理（部分接口在本地）的实现思路](https://blog.csdn.net/shop_and_sleep/article/details/122057541?spm=1001.2014.3001.5502).

# 二、Andserver 的本地文件浏览器
## 1.Andserver 的WebConfig与@Config
借用andserver文档的解释：
我们知道，每个机器上可以部署多个 Web Server，每个 WebServer 可以有不同的配置，同样的 AndServer 也提供了这样的能力，开发者可以使用Config注解来标记某个类是 Server 的配置类，这个配置总不能瞎写吧？所以Config注解的类需要实现WebConfig接口，WebConfig接口用来统一提供配置的 API。
代码如下（示例）：

```java
@Config
public class AppConfig implements WebConfig {

    @Override
    public void onConfig(Context context, Delegate delegate) {
        // 增加一个静态网站
        delegate.addWebsite(new AssetsWebsite(context, "/web"));

        // 自定义配置表单请求和文件上传的条件
        delegate.setMultipart(Multipart.newBuilder()
            .allFileMaxSize(1024 * 1024 * 20) // 单个请求上传文件总大小
            .fileMaxSize(1024 * 1024 * 5) // 单个文件的最大大小
            .maxInMemorySize(1024 * 10) // 保存上传文件时buffer大小
            .uploadTempDir(new File(context.getCacheDir(), "_server_upload_cache_")) // 文件保存目录
            .build());
    }
}
```
这样，我们就得到了一个静态的网站，通过本地的端口再加上/web就可以访问了，这并不是我们需要的。但我们已经知道通过addWebsite，技能部署一个静态网站了，那么静态文件浏览器就不再遥远。
这里值得注意的是因为SD卡也是磁盘，所以路径必须是绝对路径。

注意：FileBrowser支持热插拔。
## 2.FileBrowse（文件浏览器）
在andserver的旧版文档中，我发现了我需要的东西
FileBrowse可以部署SD卡中的内容供别人以文件的形式浏览。并且它还支持支持热插拔。

我们把上面的示例代码稍微改动一下（示例）：
```java
@Config
public class AndServerConfig implements WebConfig {
    public static String localImageUrl = "/sdcard/image/fs/img/";
    @Override
    public void onConfig(Context context, Delegate delegate) {
        File file=new File(localImageUrl);
        //文件夹不存在，则创建它
        if (!file.exists()) {
            file.mkdirs();
        }
        // 添加一个本地文件浏览器网站
        delegate.addWebsite(new FileBrowser("/sdcard/image/"));
    }
}
```
路径说明
例如我们把/sdcard目录作为部署目录：

假设我们在某个手机上部署new FileBrowse("/sdcard");目录，这个手机的本机局域网IP是192.168.1.11，我们指定AndServer监听的端口是8080。

当用户在地址栏输入http://192.168.1.11:8080时并访问后，用户将看到一个网页，网页上面会列出/sdcard下的所有文件和目录，并且在文件和目录上加上超链接。当用户点击文件时，浏览器可能会直接打开这个文件或者让用户下载这个文件；当用户点击目录时，用户将会看到一个新页面中列出了他点击的目录中的所有目录和文件。

---
至此，Andserver 配置本地文件浏览器实现webview展现本地图片的基本功能大概都实现了，如果想要实现其他的配置，例如加载Assets目录下的静态网站,Sdcard下的静态网站，你可以照猫画虎
```java
//加载Assets目录
delegate.addWebsite(new AssetsWebsite(context, "assets/web/website"));
//Sdcard
delegate.addWebsite(new StorageWebsite("/sdcard/andserver/website"));
```
甚至于自定义的http路径的静态网站

# 三、Andserver 的自定义网站
如果AndServer自带的网站不够用或者和开发者的需求有出入，那么我们可以自定义网站。
```java
public class MyWeibsite implement Website {

   ...

delegate.addWebsite(new MyWeibsite ());

```
实现Website接口，或者继承SimpleWebsite基类，然后添加进去，真的就是你想的那么简单


