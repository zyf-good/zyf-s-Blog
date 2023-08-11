---
title: 安卓的播放器对比与选型（vlc，EXOplayer，Ijkplayer，GSYVideoPlayer）📺详细过程
date: 2023-07-27 14:34:18 
tags: 音视频
categories: andorid
cover: "/images/page2.webp"
---




---

# 前言

***本文主要从实际的角度去解读和选型***
入职新公司，需要做一款涉及到播放器，播放rtsp 流的app，要求到我来选型，并给了我下面三个选择
![在这里插入图片描述](/images/player1.png)
在这之前我只是一名普普通通的安卓应用开发工程师，没有接触过音视频，如果你也和我一样，那么这件事真的太酷啦😝

然后就是为期几天的对比与选型

---
# 一、vlc
遇事不决问群友，群友给我推荐了vlc这个开源的播放器，并友好的向我推荐了文章和vlcDemo，我记不得是否是他自己的文章和demo了，这是连接

 - [安卓使用VLC播放视频，实现截图和录制功能](https://blog.csdn.net/gs12software/article/details/130618598)
 - [android 使用VLC,录像 截图功能，支持rtsp rtmp http SMB 等等。 流媒体,点播视频等等](https://blog.csdn.net/tongzhengtong/article/details/116794101?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522168923986716800197038973%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=168923986716800197038973&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-2-116794101-null-null.142%5Ev88%5Econtrol_2,239%5Ev2%5Einsert_chatgpt&utm_term=android%20vlc%20rtsp%20rtmp&spm=1018.2226.3001.4187@)

然后我浏览了[vlc的github地址](https://github.com/videolan/vlc)和[示例代码的地址](https://code.videolan.org/videolan/libvlc-android-samples)，并参考上面两个连接，实现了一个简单的vlcdemo，也对其有了一点简单的了解

**优点：**
 - 跨平台，兼容性好
- 功能全面，支持 rtsp、rtmp、ftp、http、https 等协议
- 代码完全解耦，modules 相互独立，不影响，引入新 modules 方便
 - 维护团队强大，更新频繁
 
 **缺点：**
 
- Android 平台编译出的包较大，有 16M 左右；
- 在 Android 平台上可能存在性能问题，需要优化(这个我没体会到)
- 根据群友的友好提醒，如果涉及到截图录像的功能要改so，要会c++，我不会，所以算缺点！🐶

最终因为Android 平台编译出的包较大这个缺点，没有采用vlc。😔
# 二、EXOplayer
然后就是EXOplayer的调研，什么？你问我为什么不先调研其他的，我有我自己的考量好吧，你不要教本安卓开发做事，首先我们排除mediaplayer，因为百度过后基本没有推荐的，然后就是Ijkplayer已经是一个哔哩哔哩不维护的开源项目了，而EXOplayer是谷歌开源的持续维护的，你说我先调研谁？好了，话不多说，我们骑上心爱的🛵，开始。

我先找到EXOplayer的github地址 ：[https://github.com/google/ExoPlayer](https://github.com/google/ExoPlayer)

然后找到文档：[https://exoplayer.dev/](https://exoplayer.dev/)

然后就狗血的发现啊这个EXOplayer啊，他套娃

![在这里插入图片描述](/images/player2.png)
这是什么❔，这是谷歌找安卓，老爹找儿子🐶；

然后我又去安卓官网看：[https://developer.android.google.cn/guide/topics/media/exoplayer?hl=zh_cn](https://developer.android.google.cn/guide/topics/media/exoplayer?hl=zh_cn)

![在这里插入图片描述](/images/player3.png)

好好好，你官方这么玩是吧❔，我整个一看下来，发现EXOplayer的最新依赖已经带media3的前缀了，前面几个版本还有前缀不同的代码相同的库，玩的是真滴花啊，又长见识了（更正，出现杨奇怪的场景是因为带Google前缀的exoplayer将要被废弃，新的EXOplayer被整合到media3中）

说了那么多其实都是插曲，工作赚钱嘛，不寒碜，最后我又学习并实现了用EXOplayer播放视频和自定义实现播放器界面，发现如果只是身为一个api高级调用师的话，其实使用方法都是大差不大的，这个时候我对播放器心里大概就有个底了。

再说下EXOplayer的优缺点：
**优点：**
- 接入包体积小，1.1M
- 护团队强大，更新速度快
**缺点：**
- 跨平台，不太适合直播
- 可扩展性一般，视频软解接入较麻烦
- 适合播放场景简单的项目，播放过程中无切换码流的情况

虽然说上手起来简单，包体积又小，现在还纳入了安卓官方文档，但是不太适合直播，我们的需求是实时播放一个rtsp流的视频，并且播放场景有一定复杂度，所以最后选择放弃😔

# 三、Ijkplayer
我们老规矩，首先是找github地址 ：[https://github.com/Bilibili/ijkplayer](https://github.com/Bilibili/ijkplayer)

了解过后，我同样实现了一个demo，基础使用都差不多，就是so要不然自己编译，要不然在网上找下别人编译好的，还有一件事（老爹说的不是我说的）就是有一些版本是有问题的，使用的时候最好看下，总得体验下来对于我来说就是引入的时候是最麻烦的。

**优点：**
- 包体积比 VLC 小
- 资料比较齐全（但我个人认为这也是一个缺点，到处都是问题）

**缺点：**
- 可扩展性较差，基本上没有提供 modules 供开发者二次开发
- 官方目前基本不维护，不更新

但是因为Ijkplayer支持rtsp，所以如果没有更好的选择就决定采用了，直到我发现了宝藏和本篇文章的主角**GSYVideoPlayer**

# 四、GSYVideoPlayer🔥🔥🔥
github地址： [https://github.com/CarGuo/GSYVideoPlayer](https://github.com/CarGuo/GSYVideoPlayer)

**让我们看看介绍：**
	视频播放器（IJKplayer、ExoPlayer、MediaPlayer），HTTPS支持，支持弹幕，支持滤镜、水印、gif截图，片头广告、中间广告，多个同时播放，支持基本的拖动，声音、亮度调节，支持边播边缓存，支持视频本身自带rotation的旋转（90,270之类），重力旋转与手动旋转的同步支持，支持列表播放 ，直接添加控件为封面，列表全屏动画，视频加载速度，列表小窗口支持拖动，动画效果，调整比例，多分辨率切换，支持切换播放器，进度条小窗口预览，其他一些小动画效果，rtsp、concat、mpeg。（总结，高端大气上档次）
	**让我们看看作者：**

![在这里插入图片描述](/images/player4.png)
    
    曾经有人和我说过，在中国做安卓开发不认识这个人，就不要说自己是安卓开发🐶
	
**让我们看看文档：**
![在这里插入图片描述](/images/player5.png)
现在，告诉我你们的答案！🎇🎆✨🎇🎆✨🎇🎆✨（郭神o(￣▽￣)，我爱你我要xxxxx）
咳咳，开个小小的玩笑

因为地址在这里了，要是github没有条件的可以去[https://gitee.com/CarGuo/GSYVideoPlayer](https://gitee.com/CarGuo/GSYVideoPlayer)看下文档我就不详细介绍了，我拉了项目过后自己改吧改吧，用的很满意，最终决定使用**GSYVideoPlayer**来开发项目

**优点**
- 支持好几种开源播放器，集大成者
- 可以按需引用所需要的依赖，这样一来包体积不会太大
- 作者维护很勤快，有什么问题issues，作者也会帮忙看看
- 文档写的很清楚不需要额外查资料，实在不懂代码拉下来一跑，对照着代码基本上就能理解了


**缺点：**

-有一些版本对应会有不同的问题，比如我使用的时候用了最新的依赖，按照文档不能播放rtsp流，降低了依赖过后就可以播放了

# 五、其他的开源播放器

本着学习的态度，我期间也看了一些别的开源播放器，和大家一起分享一下
## jiaozivideoplayer
[https://github.com/Jzvd/JZVideo](https://github.com/Jzvd/JZVideo)
知道这个是因为前公司的短视频播放是用饺子改的，我在app上看效果也不是很好，原本好像叫节操，现在改名叫饺子，网友都说难用，所以没有详细尝试，感兴趣的兄弟可以去看看
## MediaPlayer
[https://developer.android.google.cn/guide/topics/media/mediaplayer?hl=zh_cn](https://developer.android.google.cn/guide/topics/media/mediaplayer?hl=zh_cn)
因为原生的MediaPlayer不支持rtsp流，所以我也没有过多的看

## QPlayer2
[https://github.com/pili-engineering/QPlayer2-Android](https://github.com/pili-engineering/QPlayer2-Android)

七牛播放器的衍生品，原本的七牛播放器已经停止维护，也是因为不支持rtsp流所以没有尝试

## SmarterStreaming
[https://github.com/daniulive/SmarterStreaming](https://github.com/daniulive/SmarterStreaming)

大牛直播，看起来很牛逼的样子，可惜要收费

---
# 总结
在我这个需求下我最终选用了GSYVideoPlayer，但各位朋友们还是要看自己的需求，选用自己适合的三方框架，开发起来才能事半功倍
# 参考

[Ijkplayer、ExoPlayer、VLC播放器综合比较](https://juejin.cn/post/6956604648476622856?searchId=2023072616491758248039F36B28127672#comment)
