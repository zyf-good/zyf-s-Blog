---
title: Android Kotlin 协程（coroutine）入门篇
date:  2022-03-24 17:54:17
tags: Kotlin协程（coroutine）
categories: andorid
cover: "/images/android3.jpeg"
---

---

# 前言
`ps：文章会很长`

协同程序（coroutine﻿）实在是太好用了，所以我们来学习吧！本篇文章会通过解读kotlin官方文档和android  development文档来一步一步认识Kotlin 协程，没有Kotlin 基础的小伙伴建议先学习下 Kotlin 哦。

---


# 一、协程基础
本节介绍协程基本的概念。下面这些都是kotlin官方的例子，非常经典。
>依赖和权限

```kotlin
//依赖
 implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.9")
      //retrofit
    api 'com.squareup.retrofit2:retrofit:2.6.0'
    api 'com.squareup.retrofit2:converter-gson:2.6.0'
    api 'com.squareup.retrofit2:adapter-rxjava2:2.6.0'
    api 'com.trello.rxlifecycle2:rxlifecycle-components:2.6.1'
    api 'com.facebook.stetho:stetho:1.5.0'
    api 'com.facebook.stetho:stetho-okhttp3:1.5.0'

 //权限
 <!-- 访问网络的权限 -->
    <uses-permission android:name="android.permission.INTERNET"></uses-permission>
```



## 第一个协程
协程是可暂停计算的一个实例。它在概念上类似于线程，它需要一个代码块来运行，该代码块与其余代码同时工作。然而，协程并不绑定到任何特定的线程。它可以在一个线程中暂停执行，在另一个线程中恢复。
协同程序可以被认为是轻量级的线程，但是有许多重要的区别，使得它们在现实生活中的使用与线程非常不同。
下面这个代码示例将是我们的第一个协程

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking { // this: CoroutineScope  
	// launch a new coroutine and continue  启动一个新的协程并连接
    launch { 
    // non-blocking delay for 1 second (default time unit is ms)延时一秒
        delay(1000L) 
        // print after delay 延时后打印
        println("World!") 
    }
    // 主线程中的"Hello"继续执行，而"World!"延时后打印
    println("Hello") 
}
```
不出意料，我们会得到这样的结果：
>Hello
World!

从这个结果来看，这确实和线程很像不是吗？下面我们来分析上面的代码

***launch***
是一个协程构建者。它与代码的其余部分同时启动一个新的协程，该程序将继续独立工作。这就是为什么Hello首先被打印出来。
***delay***
是一种特殊的暂停功能。它会在特定时间暂停协程。挂起协程，但不会阻止底层线程，允许其他协程运行并将底层线程用于其代码。可以与线程中的sleep()相互印证以便记忆（并不是说可以等价哦）
***runBlocking***
也是一个协程构建器，它将常规的fun main（）的非协程与runBlocking{…}内部的协同程序代码连接起来。在IDE中，在runBlocking开头的花括号后面有一个提示：CoroutineScope。

`注意：如果在这段代码中删除或忘记了runBlocking，将在启动调用中得到如下错误，因为启动只在CoroutineScope中声明：`

```kotlin
Unresolved reference: launch
```
runBlocking的名称意味着运行它的线程（在本例中是主线程）在调用期间被阻塞，直到runBlocking{…}中的所有协程被阻塞完成他们的代码逻辑。您经常会看到在最上层的应用程序像这样使用runBlocking，而在真正的代码中很少使用，因为线程是昂贵的资源，阻塞它们是低效的，而且通常是不需要的。

###  结构化并发
协同路由遵循结构化并发原则，这意味着新的协同路由只能在限定协同路由生存期的特定协同路由作用域中启动。上面的例子显示runBlocking建立了相应的作用域，这就是为什么前面的例子要等到World！延迟一秒钟后打印，然后退出。

在实际应用程序中，将启动许多协同程序。结构化并发确保它们不会丢失，也不会泄漏。外部作用域在其所有子协同程序完成之前无法完成。结构化并发还可以确保正确报告代码中的任何错误，并且永远不会丢失。

## 挂起函数 suspend 
上诉仅仅是一个最基础的协同程序，我们可以提取launch{…}中的代码块变成一个单独的函数。当你对这段代码执行“提取函数”重构时（就是alt+enter啦），你会得到一个带有suspend修饰符的新函数（挂起函数）。这是你的第一个暂停功能。挂起函数可以像常规函数一样在协同路由中使用，也可以反过来使用其他挂起函数（如本例中的延迟）来挂起协同路由的执行。

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking { // this: CoroutineScope
    launch { doWorld() }
    println("Hello")
}

// this is your first suspending function
//第一个挂起函数
suspend fun doWorld() {
    delay(1000L)
    println("World!")
}
```
同样得到如下结果
>Hello
World!

## 范围构造器 Scope builder
除了不同构建器提供的协同程序作用域之外，还可以使用协同程序作用域构建器声明自己的作用域。它创建了一个协程作用域，直到所有启动的子项都完成后才会完成。

runBlocking和coroutineScope的构建者可能看起来很相似，因为他们都在等待自己的主体和所有子项的完成。主要区别在于runBlocking方法阻塞当前线程等待，而coroutineScope只是暂停，释放底层线程供其他使用。由于这种差异，runBlocking是一个常规函数，而coroutineScope是一个挂起函数。

我们可以从任何挂起函数使用coroutineScope。例如，可以将Hello和World的并发打印移动到suspend fun doWorld（）函数中：

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    doWorld()
}

suspend fun doWorld() = coroutineScope {  // this: CoroutineScope
    launch {
        delay(1000L)
        println("World!")
    }
    println("Hello")
}
```
同样的结果：
>Hello
World!

## 范围构造器 Scope builder 和 并发 concurrency﻿
范围构造器 可以在任何挂起函数中使用，以执行多个并发操作。让我们在doWorld挂起函数中启动两个并发协同路由：

```kotlin
import kotlinx.coroutines.*

// Sequentially executes doWorld followed by "Done"
//顺序执行
fun main() = runBlocking {
    doWorld()
    println("Done")
}

// Concurrently executes both sections
//同时执行两个部分
suspend fun doWorld() = coroutineScope { // this: CoroutineScope
    launch {
        delay(2000L)
        println("World 2")
    }
    launch {
        delay(1000L)
        println("World 1")
    }
    println("Hello")
}
```
launch {…}中的两段代码块同时执行，从开始一秒后，首先打印World 1，从开始两秒后打印World 2。doWorld中的一个coroutineScope只有在两者都完成后才能完成，因此doWorld返回并允许仅在完成后打印Done字符串：
>Hello
World 1
World 2
Done

## job
 launch协程构造器返回一个job对象，该对象是启动协同程序的句柄，可用于显式等待其完成。例如，您可以等待子协同程序完成，然后打印“Done”字符串：
 

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = launch { 
    // launch a new coroutine and keep a reference to its Job
    //启动一个新的协同程序，并保留对其工作的引用
        delay(1000L)
        println("World!")
    }
    println("Hello")
    // wait until child coroutine completes
    //等待子协同程序完成
    job.join() 
    println("Done")     
}
```
结果如下：
>Hello
World!
Done
## 轻量级
我们有如下示例代码：

```kotlin
import kotlinx.coroutines.*

//sampleStart
fun main() = runBlocking {
    repeat(100_000) { // launch a lot of coroutines
        launch {
            delay(5000L)
            print(".")
        }
    }
}
//sampleEnd
```
它会启动10万个协程，5秒后，每个协同程序都会打印一个点。
但是，如果我们用线程试试（删除runBlocking，用thread替换launch，用thread.sleep替换delay）。会发生什么？（***代码很可能会产生某种内存不足错误***）

# 二、Android 上的 Kotlin 协程
Android的开发文档中是这么介绍协程的：
协程是一种并发设计模式，您可以在 Android 平台上使用它来简化异步执行的代码。协程是在版本 1.3 中添加到 Kotlin 的，它基于来自其他语言的既定概念。

在 Android 上，协程有助于管理长时间运行的任务，如果管理不当，这些任务可能会阻塞主线程并导致应用无响应。使用协程的专业开发者中有超过 50% 的人反映使用协程提高了工作效率。本主题介绍如何使用 Kotlin 协程解决以下问题，从而让您能够编写出更清晰、更简洁的应用代码。


## 特点
协程是我们在 Android 上进行异步编程的推荐解决方案。值得关注的特点包括：
 - **轻量**：您可以在单个线程上运行多个协程，因为协程支持挂起，不会使正在运行协程的线程阻塞。挂起比阻塞节省内存，且支持多个并行操作。
 -  **内存泄漏更少**：使用结构化并发机制在一个作用域内执行多项操作。
 -  **内置取消支持**：取消操作会自动在运行中的整个协程层次结构内传播。
 - **Jetpack 集成**：许多 Jetpack 库都包含提供全面协程支持的扩展。某些库还提供自己的协程作用域，可供您用于结构化并发。

## Android 上 Kotlin 协程+Retrofit进行网络请求的示例
为了方便对比，我们先使用单独的Retrofit，如果对Retrofit不熟悉的同学可以参考笔者的这篇文章
>java版本的,支持协程的Retrofit版本是至少要到大于2.6哦，本文使用的就是2.6版本的Retrofit
[android retrofit 从无知到入门（最新最全）](https://blog.csdn.net/shop_and_sleep/article/details/123526236?spm=1001.2014.3001.5502)
###  一些代码的准备
>API

```kotlin
import retrofit2.Call
import retrofit2.Response
import retrofit2.http.GET

interface ISystemRemoteService {
    @get:GET("/system/alive")
    val systemStatus: Call<Response<Any>>
}
```
>RetrofitService

```kotlin
import android.util.Log
import retrofit2.Retrofit
import okhttp3.OkHttpClient
import com.zyf.kotlindemo.HttpsUtils
import kotlin.Throws
import com.zyf.kotlindemo.RetrofitService
import java.util.concurrent.TimeUnit

class RetrofitService private constructor() {
    private val retrofit: Retrofit
    fun <T> create(clazz: Class<T>?): T {
        return retrofit.create(clazz)
    }

    companion object {
        private val client = OkHttpClient.Builder()
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .sslSocketFactory(HttpsUtils.getSslSocketFactory(null, null, null))
            .hostnameVerifier { hostname, session -> true }
            .writeTimeout(30, TimeUnit.SECONDS).addInterceptor { chain ->
                val builder = chain.request()
                    .newBuilder()
                val request = builder.build()
                chain.proceed(request)
            }.build()

        private var i: RetrofitService? = null
        fun i(): RetrofitService? {
            if (i == null) {
                i = RetrofitService()
            }
            return i
        }

        fun rebuild() {
            i = RetrofitService()
        }
    }

    init {
        val serverUrl = "https://blog.csdn.net/shop_and_sleep"
        retrofit = Retrofit.Builder()
            .baseUrl(serverUrl) //设置超时时间,intercept
            .client(client)
            .build()
        Log.i("ServerUrl:------", serverUrl)
    }
}
```

###  Kotlin 简单使用Retrofit

>只使用Retrofit不使用协程

```kotlin
 val api = RetrofitService.i()?.create(ISystemRemoteService::class.java)
        val call = api?.systemStatus
        //这里因为我是在主线程，所以只能异步
        val result = call?.enqueue(object : Callback<Response<Any>>{
            override fun onResponse(call: Call<Response<Any>>, response: Response<Response<Any>>) {
                //成功的回调
            }

            override fun onFailure(call: Call<Response<Any>>, t: Throwable) {
                //失败的回调
            }

        })
```
###  Kotlin android 简单使用协程
在第一个协程中我们知道每一个协程都要有一个范围构造器，那么在安卓中之前的第一个协程就可以变为

```kotlin
        CoroutineScope(Dispatchers.Main).launch {
            // non-blocking delay for 1 second (default time unit is ms)延时一秒
            delay(1000L)
            // print after delay 延时后打印
            println("World!")
        }
        println("Hello")
```
***Dispatchers.Main***
代表在主线程中开启这个协程，Dispatchers是一个调度器，除了Main，在他的源码中我们还可以看到其他的，以下面这个表格来阐述：
|  分组|作用  |
|--|--|
| Default | 默认的调度器（如果不写得话默认就是这个） |
| Main | 主线程的调度器（如果不写得话默认就是这个） |
| Unconfined| 一个不局限于任何特定线程的协同程序调度程序|
| IO | 顾名思义设计用于将阻塞IO任务卸载到共享线程池的调度器） |

###  Kotlin android 简单使用协程+retrofit
了解了调度器Dispatchers，协程构造器，和协程构造器的返回job后，一个简单的网络请求就顺其自然的出现了：

```kotlin
    CoroutineScope(Dispatchers.Main).launch {

            /*因为第一个协程还是主线程，
            Android规定网络请求必须在子线程,
            所以这里我们通过withContext获取请求结果,
            通过调度器Dispatcher切换到IO线程,
            这个操作会挂起当前协程,但是不会阻塞当前线程*/
            val result = withContext(Dispatchers.IO) {
                /*这里已经是在子线程了,
            	所以使用execute()同步请求
          		withContext会把闭包最后一行代码的返回值返回出去
          		所以上面的result就是Response类型*/
                RetrofitService.i()?.create(ISystemRemoteService::class.java)?.
                systemStatus?.
                execute()?.
                body()
            }
            Log.d("onCreate", "result: "+result.toString())
            if (result?.isSuccessful == true){
                Toast.makeText(mContext, "请求成功", Toast.LENGTH_SHORT).show()
            }
        }
```

> 其中 ***withContext*** 使用给定的协同路由上下文调用指定的挂起块，挂起直到完成，然后返回，除了 ***launch*** 
> ，***withContext***，还有 ***async*** ，由于篇幅原因不多赘述，有兴趣可以自行了解；

 值得一提的是，如果调用了***withContext***，在其调度器开始执行代码时被取消，它将丢弃'withContext'的结果并抛出[CancellationException]。
 为了解决这个问题，就不得不提协程的内置取消支持，所以我们的完整代码变成了如下：
 

```kotlin
import android.support.v7.app.AppCompatActivity
import android.os.Bundle
import android.util.Log
import android.view.Gravity
import android.widget.Button
import android.widget.Toast
import kotlinx.android.synthetic.main.activity_main.*
import kotlinx.coroutines.*
import retrofit2.Call
import retrofit2.Callback
import retrofit2.Response
import kotlin.coroutines.CoroutineContext


class MainActivity : AppCompatActivity() ,CoroutineScope {
    private val mContext by lazy { this } //（这里使用了委托，表示只有使用到mContext 才会执行该段代码）
    //job用于控制协程,后面launch{}启动的协程,返回的job就是这个job对象
    private lateinit var job: Job
    //继承CoroutineScope必须初始化coroutineContext变量
    // 这个是标准写法,+其实是plus方法前面表示job,用于控制协程,后面是Dispatchers,指定启动的线程
    override val coroutineContext: CoroutineContext
        get() = job + Dispatchers.Main


   override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        job = Job()
        setContentView(R.layout.activity_main)
        CoroutineScope(Dispatchers.Main).launch {

      
             RetrofitService.i()?.create(ISystemRemoteService::class.java)?.
                systemStatus?.
                execute()?.
                body()
            }
            Log.d("onCreate", "result: "+result.toString())
            if (result?.isSuccessful == true){
                Toast.makeText(mContext, "请求成功", Toast.LENGTH_SHORT).show()
            }
        }
    }
    
    override fun onDestroy() {
    	//当acitivity结束之后,我们不需要再请求网络了,结束当前协程
        job.cancel()
        super.onDestroy()

    }
}
```
这个简单示例到此就结束了，但实际项目一般会在viewmodel中使用网络请求而不是在activity中，这就不得不提协程的另一个优点Jetpack 集成了，如果是在viewmodel中，可以使用viewModelScope来绑定viewModel的生命周期，否则就要像示例那样手动调用 job.cancel()方法

---

# 参考
[kotlin官方文档](https://www.kotlincn.net/docs/reference/)
[安卓官方文档](https://developer.android.google.cn/kotlin)
[协程配合retrofit调用网络请求](https://www.cnblogs.com/sw-code/p/14451921.html)
[Android中使用Kotlin协程(Coroutines)和Retrofit进行网络请求(一)](https://blog.csdn.net/weixin_44407870/article/details/86767075)

