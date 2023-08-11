---
title: Android Socket 简单介绍
date: 2023-08-11 14:54:55
tags: Socket
categories: andorid
cover: "/images/page3.webp"
---



---

# 前言
最近需求需要使用Socket进行通讯，我在工作后的安卓开发中没有接触过，所以有了这篇文章；
写的时候想起来好像上大学的时候学过，后面一直没用忘记了；
之前公司同事的男朋友也在写这个来着，我当时还嘲笑怎么还会有人用这个，现在自己遇到了🐕；

---
# 一、Socket是什么？
## 百度百科的解释
所谓套接字(Socket)，就是对网络中不同主机上的应用进程之间进行双向通信的端点的抽象。一个套接字就是网络上进程通信的一端，提供了应用层进程利用网络协议交换数据的机制。从所处的地位来讲，套接字上联应用进程，下联网络协议栈，是应用程序通过网络协议进行通信的接口，是应用程序与网络协议栈进行交互的接口。

套接字Socket=（IP地址：端口号），套接字的表示方法是点分十进制的lP地址后面写上端口号，中间用冒号或逗号隔开。每一个传输层连接唯一地被通信两端的两个端点（即两个套接字）所确定。例如：如果IP地址是210.37.145.1，而端口号是23，那么得到套接字就是(210.37.145.1:23)

## 我自己的理解
分客户端与服务端，socket以服务端的ip地址和约定好的端口号进行组合得到套接字进行一个两端的相互连接，达到能够互相收发消息的一个目的。


# 二、简单示例
下面的代码将演示如何在一个app中实现服务端和客户端的简单示例
## 1.服务端
先创建抽象的**ServerCallback**

```kotlin
interface ServerCallback {
    //接收客户端的消息
    fun receiveClientMsg(success: Boolean, msg: String)
    //其他消息，例如有客户端连接和发送消息成功等
    fun otherMsg(msg: String)
}
```

然后创建**SocketServer**来管理服务端的连接

代码如下：
```kotlin
import android.util.Log
import java.io.IOException
import java.io.InputStream
import java.io.OutputStream
import java.net.ServerSocket
import java.net.Socket

object SocketServer {

    private val TAG = SocketServer::class.java.simpleName

    private const val SOCKET_PORT =
        7878

    private var socket: Socket? = null
    private var serverSocket: ServerSocket? = null

    private lateinit var mCallback: ServerCallback

    private lateinit var outputStream: OutputStream

    var result = true
    /**
     * 开启服务
     */
    fun startServer(callback: ServerCallback): Boolean {
        Log.i(TAG, "startServer: ")
        mCallback = callback
        Thread {
            try {
                serverSocket = ServerSocket(SOCKET_PORT)
                while (result) {
                    socket = serverSocket?.accept()
                    mCallback.otherMsg("${socket?.inetAddress} to connected")
                    ServerThread(socket!!, mCallback).start()
                }
            } catch (e: IOException) {
                e.printStackTrace()
                result = false
            }
        }.start()
        return result
    }

    /**
     * 关闭服务
     */
    fun stopServer() {
        Log.i(TAG, "stopServer: ")
        socket?.apply {
            shutdownInput()
            shutdownOutput()
            close()
        }
        serverSocket?.close()
    }

    /**
     * 发送到客户端
     */
    fun sendToClient(msg: String) {
        Thread {
            if (socket!!.isClosed) {
                Log.e(TAG, "sendToClient: Socket is closed")
                return@Thread
            }
            outputStream = socket!!.getOutputStream()
            try {
                outputStream.write(msg.toByteArray())
                outputStream.flush()
                mCallback.otherMsg("toClient: $msg")
                Log.d(TAG, "发送到客户端成功")
            } catch (e: IOException) {
                e.printStackTrace()
                Log.e(TAG, "向客户端发送消息失败")
            }
        }.start()
    }

    class ServerThread(private val socket: Socket, private val callback: ServerCallback) :
        Thread() {

        override fun run() {
            val inputStream: InputStream?
            try {
                inputStream = socket.getInputStream()
                val buffer = ByteArray(1024)
                var len: Int
                var receiveStr = ""
                if (inputStream.available() == 0) {
                    Log.e(TAG, "inputStream.available() == 0")
                }
                while (inputStream.read(buffer).also { len = it } != -1) {
                    receiveStr += String(buffer, 0, len, Charsets.UTF_8)
                    if (len < 1024) {
                        callback.receiveClientMsg(true, receiveStr)
                        receiveStr = ""
                    }
                }
            } catch (e: IOException) {
                e.printStackTrace()
                e.message?.let { Log.e("socket error", it) }
                callback.receiveClientMsg(false, "")
            }
        }
    }
}

```
可以看到我们在通过startServer方法初始化ServerSocket时，只使用到了SOCKET_PORT，这个ServerSocket方法接收一个端口号，不接收ip，因为它将作为服务端
**ServerSocket方法的注释译文：** 创建绑定到指定端口的服务器套接字。端口号为0表示端口号是自动分配的，通常是从临时端口范围分配的。然后可以通过调用getLocalPort来检索这个端口号。传入连接指示(连接请求)的最大队列长度设置为50。如果连接指示在队列已满时到达，则拒绝连接。
## 2.客户端

同样我们创建一个抽象的 **ClientCallback**


```kotlin
interface ClientCallback {
    //接收服务端的消息
    fun receiveServerMsg(msg: String)
    //其他消息
    fun otherMsg(msg: String)
}
```

然后创建SocketClient来管理服务端的连接

代码如下：

```kotlin


import android.util.Log
import java.io.IOException
import java.io.InputStream
import java.io.InputStreamReader
import java.io.OutputStream
import java.net.Socket
import java.util.Locale

object SocketClient {

    private val TAG = SocketClient::class.java.simpleName

    private var socket: Socket? = null

    private var outputStream: OutputStream? = null

    private var inputStreamReader: InputStreamReader? = null

    private lateinit var mCallback: ClientCallback

    private const val SOCKET_PORT = 7878

    /**
     * 连接服务
     */
    fun connectServer(ipAddress: String, callback: ClientCallback) {
        mCallback = callback
        Thread {
            try {
                socket = Socket(ipAddress, SOCKET_PORT)
                ClientThread(socket!!, mCallback).start()
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }.start()
    }

    /**
     * 关闭连接
     */
    fun closeConnect() {
        try {
            inputStreamReader?.close()
            outputStream?.close()
            socket?.apply {
                shutdownInput()
                shutdownOutput()
                close()
            }
            Log.d(TAG, "关闭连接")
        }catch (e: Exception){
            e.printStackTrace()
        }

    }

    /**
     * 发送数据至服务器
     * @param msg 要发送至服务器的字符串
     */
    fun sendToServer(msg: String) {
        Thread {
            if (socket!!.isClosed) {
                Log.e(TAG, "sendToServer: Socket is closed")
                return@Thread
            }
            outputStream = socket?.getOutputStream()
            try {
                outputStream?.write(msg.toByteArray())
                outputStream?.flush()
                mCallback.otherMsg("toServer: $msg")
                Log.e(TAG, "向服务端发送消息成功")
            } catch (e: IOException) {
                e.printStackTrace()
                Log.e(TAG, "向服务端发送消息失败")
            }
        }.start()
    }



    class ClientThread(private val socket: Socket, private val callback: ClientCallback) : Thread() {
        override fun run() {
            val inputStream: InputStream?
            try {
                inputStream = socket.getInputStream()
                val buffer = ByteArray(1024)
                var len: Int
                var receiveStr = ""
                if (inputStream.available() == 0) {
                    Log.e(TAG, "inputStream.available() == 0")
                }
                while (inputStream.read(buffer).also { len = it } != -1) {
                    receiveStr += String(buffer, 0, len, Charsets.UTF_8)
                    if (len < 1024) {
                        callback.receiveServerMsg(receiveStr)
                        receiveStr = ""
                    }
                }
            } catch (e: IOException) {
                e.printStackTrace()
                e.message?.let { Log.e("socket error", it) }
                callback.receiveServerMsg( "")
            }
        }
    }
}

```
可以看到我们在客户端通过connectServer方法连接服务端时，使用了Socket方法，它接收了一个ip和一个约定好的端口号，这里的ip就是服务端的ip
**Socket方法的注释译文为：** 创建流套接字并将其连接到指定主机上的指定端口号。如果指定的主机为空，则相当于指定地址为InetAddress。getByName (null)。也就是说，它相当于指定loopback接口的地址。


## 3.布局

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:orientation="vertical">


    <LinearLayout
        android:id="@+id/lay_server"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">
        <TextView
            android:id="@+id/tv_ip_address"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:padding="16dp"
            android:text="Ip地址：" />

        <Button
            android:id="@+id/btn_start_service"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginStart="16dp"
            android:layout_marginEnd="16dp"
            android:text="开启服务"
             />
        <Button
            android:id="@+id/btn_connect_service"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginStart="16dp"
            android:layout_marginEnd="16dp"
            android:text="开启链接"
             />
        <Button
            android:id="@+id/btn_client_send_msg"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginStart="16dp"
            android:layout_marginEnd="16dp"
            android:text="客户端发送消息"
            />
        <Button
            android:id="@+id/btn_server_send_msg"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginStart="16dp"
            android:layout_marginEnd="16dp"
            android:text="服务端发送消息"
            />
    </LinearLayout>

</LinearLayout>
```

## 4.实现
在activity中

```kotlin
import android.net.wifi.WifiManager
import android.os.Bundle
import android.util.Log
import androidx.appcompat.app.AppCompatActivity
import com.zyf.vlc.databinding.ActivityMainBinding
import com.zyf.vlc.socket.ClientCallback
import com.zyf.vlc.socket.ServerCallback
import com.zyf.vlc.socket.SocketClient
import com.zyf.vlc.socket.SocketServer


class MainActivity : AppCompatActivity(),ServerCallback, ClientCallback {

    private  val TAG = "MainActivity"
    private lateinit var binding: ActivityMainBinding

    //Socket服务是否打开
    private var openSocket = false

    private var connectSocket = false

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)


        binding.tvIpAddress.text = "Ip地址：${getIp()}"

        //开启服务/关闭服务 服务端处理
        binding.btnStartService.setOnClickListener {
            openSocket = if (openSocket) {
                SocketServer.stopServer();false
            } else SocketServer.startServer(this)

            //改变按钮文字
            binding.btnStartService.text = if (openSocket) "关闭服务" else "开启服务"
        }

        binding.btnConnectService.setOnClickListener {
            val ip = getIp()
            connectSocket = if (connectSocket) {
                SocketClient.closeConnect();false
            } else {
                SocketClient.connectServer(ip, this);true
            }

            binding.btnConnectService.text = if (connectSocket) "关闭连接" else "连接服务"
        }

        binding.btnClientSendMsg.setOnClickListener {
            SocketClient.sendToServer("客户端发送消息")
        }


        binding.btnServerSendMsg.setOnClickListener {
            SocketServer.sendToClient("服务端发送消息")
        }


 private fun getIp() =
        intToIp((applicationContext.getSystemService(WIFI_SERVICE) as WifiManager).connectionInfo.ipAddress)

    private fun intToIp(ip: Int) =
        "${(ip and 0xFF)}.${(ip shr 8 and 0xFF)}.${(ip shr 16 and 0xFF)}.${(ip shr 24 and 0xFF)}"



    override fun receiveClientMsg(success: Boolean, msg: String) {
        Log.i(TAG, "receiveClientMsg: $msg")
    }

    override fun receiveServerMsg(msg: String) {
        Log.i(TAG, "receiveServerMsg: $msg")
    }

    override fun otherMsg(msg: String) {
        Log.i(TAG, "otherMsg: $msg")
    }
```


最终实现的效果如下

![在这里插入图片描述](/images/socket1.png)
运行后依次点击按钮，可以得到如下日志
![在这里插入图片描述](/images/socket2.png)


---

# 参考
[Android Socket通讯](https://blog.csdn.net/qq_38436214/article/details/126177462)
# 总结
本文简单介绍了Android 中Socket 的使用方法，并通过简单的示例帮助理解。
可以关注下[我的公众号](http://weixin.qq.com/r/kRyxqZbENMtLrdLK90mD)，我会不定时分享文章和帮助解决问题