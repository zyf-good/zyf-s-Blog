---
title: ComposeDesktop实战宝可梦图鉴
date: 2023-06-14 16:13:36
tags: Jetpack Compose
categories: andorid
cover: "/images/page1.webp"
---


---

# 前言
  阅读本文需要一定compose基础，如果没有请移步[Jetpack Compose入门详解（实时更新）](https://blog.csdn.net/shop_and_sleep/article/details/124569994?spm=1001.2014.3001.5501)
  
接口数据来源于[pokeapi](https://pokeapi.co/)

[项目源代码](https://github.com/zyf-good/pokemon2/tree/master)   
如果你觉得不错，请给我一个star，THKS
---
# 实现效果
闲话不多说，让我们来看看实现效果
![在这里插入图片描述](/images/desktop.png)         
可以看到我们实现了一个非常简洁的宝可梦图鉴，展示了所有世代的宝可梦，并提供了搜索，与宝可梦图鉴app的UI有了一些变化。

---
# 一、架构介绍
如图，展示了主要的一些文件：
![在这里插入图片描述](/images/image111.png)         
本项目没有适配安卓端，iOS端和web端（如果有乐于开源的小伙伴可以联系我），因此所有核心代码都在desktop中

 - http  - 接口 
 - util -工具类
 - ui -Compose组件           
 - viewmodels - viewmodel

# 二、一些的功能点的介绍

##  PreCompose 
通过[PreCompose](https://github.com/Tlaster/PreCompose)        这个三方库，可以像写app一样写desktop，当然这里面有一些与app使用有些差别

```kotlin
import androidx.compose.ui.unit.DpSize       
import androidx.compose.ui.window.application
import androidx.compose.ui.window.rememberWindowState
import moe.tlaster.precompose.PreComposeWindow
import ui.FullScreenView
import util.EnvUtil


fun main() = application {
    val windowState = rememberWindowState(size = DpSize(AppConfig.windowMinWidth, AppConfig.windowMinHeight))
    PreComposeWindow(
        state = windowState,
        onCloseRequest = ::exitApplication,
        undecorated = EnvUtil.isWindows(),
        title = ""
    ) {
        //window.minimumSize = Dimension(AppConfig.windowMinWidth.value.toInt(), AppConfig.windowMinHeight.value.toInt())
        window.rootPane.apply {
            rootPane.putClientProperty("apple.awt.fullWindowContent", true)
            rootPane.putClientProperty("apple.awt.transparentTitleBar", true)
            rootPane.putClientProperty("apple.awt.windowTitleVisible", false)
        }
        FullScreenView()
        //CpnWindowsPlaformDecoratedButtons(windowState)
    }
}
```
PreComposeWindow就是必备的，不然后续使用所有的与PreCompose相关的都会报错
例如初始化viewmodel

```kotlin
import moe.tlaster.precompose.ui.viewModel  

val viewModel  = viewModel { PokemonListViewModel() }
```


##  加载网络图片
桌面端加载网络图也没有安卓的原生库使用，这里使用了[compose-desktop-imageloader](https://github.com/succlz123/compose-desktop-imageloader)       

```kotlin
package ui.common             

import androidx.compose.foundation.Image 
import androidx.compose.runtime.Composable 
import androidx.compose.ui.Modifier 
import androidx.compose.ui.layout.ContentScale 
import androidx.compose.ui.res.painterResource 
import org.succlz123.lib.imageloader.ImageAsyncImageFile 
import org.succlz123.lib.imageloader.ImageAsyncImageUrl 
import org.succlz123.lib.imageloader.ImageRes 
import org.succlz123.lib.imageloader.core.ImageCallback 


@Composable
fun AsyncImage( 
    modifier: Modifier, 
    url: String?, 
    placeHolderUrl: String? = "image/ic_pokmon.jpeg", 
    errorUrl: String? = "image/ic_pokmon.jpeg", 
    contentScale: ContentScale = ContentScale.Crop 
) {
    val imgUrl = url ?: "image/ic_pokmon.jpeg" 
    if (imgUrl.startsWith("http")) { 
        ImageAsyncImageUrl(url = imgUrl, imageCallback = ImageCallback(placeHolderView = { 
            placeHolderUrl?.let { 
                Image( 
                    painter = painterResource(placeHolderUrl), 
                    contentDescription = imgUrl, 
                    modifier = modifier, 
                    contentScale = contentScale 
                )
            }
        }, errorView = { 
            errorUrl?.let { 
                Image( 
                    painter = painterResource(errorUrl), 
                    contentDescription = imgUrl, 
                    modifier = modifier, 
                    contentScale = contentScale 
                )
            }
        }) {
            Image( 
                painter = it, contentDescription = imgUrl, modifier = modifier, contentScale = contentScale 
            )
        })
    } else if (imgUrl.startsWith("/") || imgUrl.contains(":\\")) { 
        ImageAsyncImageFile(filePath = imgUrl, imageCallback = ImageCallback(placeHolderView = { 
            placeHolderUrl?.let { 
                Image( 
                    painter = painterResource(placeHolderUrl), 
                    contentDescription = imgUrl, 
                    modifier = modifier, 
                    contentScale = contentScale 
                )
            }
        }, errorView = { 
            errorUrl?.let { 
                Image( 
                    painter = painterResource(errorUrl), 
                    contentDescription = imgUrl, 
                    modifier = modifier, 
                    contentScale = contentScale 
                )
            }
        }) {
            Image( 
                painter = it, contentDescription = imgUrl, modifier = modifier, contentScale = contentScale 
            )
        })
    } else { 
        ImageRes(resName = imgUrl, imageCallback = ImageCallback(placeHolderView = { 
            placeHolderUrl?.let { 
                Image( 
                    painter = painterResource(placeHolderUrl), 
                    contentDescription = imgUrl, 
                    modifier = modifier, 
                    contentScale = contentScale 
                )
            }
        }, errorView = { 
            errorUrl?.let { 
                Image( 
                    painter = painterResource(errorUrl), 
                    contentDescription = imgUrl, 
                    modifier = modifier, 
                    contentScale = contentScale 
                )
            }
        }) {
            Image( 
                painter = it, contentDescription = imgUrl, modifier = modifier, contentScale = contentScale 
            )
        })
    }
}
```


# 致谢
项目中大部分技术栈都参考自[NCMusicDesktop](https://github.com/sskEvan/NCMusicDesktop)，感谢大佬