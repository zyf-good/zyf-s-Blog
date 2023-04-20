---
title: Jetpack Compose 实战 宝可梦图鉴
date: 2023-04-13 11:14:46 
tags: Jetpack Compose
categories: andorid
cover: "/images/andorid1.jpg"
---



---

# 前言
  阅读本文需要一定compose基础，如果没有请移步[Jetpack Compose入门详解（实时更新）](https://blog.csdn.net/shop_and_sleep/article/details/124569994?spm=1001.2014.3001.5501)
  学Compose学了有小半年的时间了，一直都是看官方的一些基础的教程并总结学习。最近终于实战了一个宝可梦图鉴的小项目，麻雀虽小五脏俱全。除了Compose外，还使用了一下一些Jetpack组件
 
* [accompanist-pager](https://google.github.io/accompanist/pager/)

* [coil](https://github.com/coil-kt/coil)

* [hilt](https://developer.android.google.cn/training/dependency-injection/hilt-jetpack#compose)

* [paging3](https://developer.android.google.cn/topic/libraries/architecture/paging/v3-overview)

* [navigation-compose](https://developer.android.google.cn/jetpack/compose/navigation)

* [accompanist-systemuicontroller](https://google.github.io/accompanist/systemuicontroller/)

* [Retrofit](https://square.github.io/retrofit/) 

* [Kotlin Flow](https://developer.android.com/kotlin/flow) 

接口数据来源于[pokeapi](https://pokeapi.co/)

[项目源代码](https://github.com/zyf-good/pokemon)
如果你觉得不错，请给我一个star，THKS

---
# 实现效果
闲话不多说，让我们来看看实现效果
![在这里插入图片描述](/images/Pokmon1.gif)
可以看到我们实现了一个非常简洁的宝可梦图鉴，展示了所有世代的宝可梦，并提供了搜索，点击进入详情查看他们的属性

# 一、架构介绍
如图，展示了主要的一些文件：
![在这里插入图片描述](/images/Pokmon2.png)

 - api   - 接口
 - nav - navigation导航
 - utils -工具类
 - view -Compose组件
 - viewmodels - viewmodel

# 二、一些的功能点的介绍
主要是开发过程中遇到的一些问题
## 加载图片并获取主色,再讲主色设置为背景
展示图片本来有封装好的***CommonImage.kt***用来展示图片，但是在列表中每一张图片的背景色都是动态获取的，当时做的时候觉得比较难弄，后面实现了过后就觉得还好啦。

引入的库

```kotlin
    implementation "io.coil-kt:coil-compose:2.1.0"
        //get Dominant image color
    implementation 'androidx.palette:palette-ktx:1.0.0'
```
主要代码在**FullScreenView.kt**中的 **item()** 中

```kotlin
	//我们所动态获取的图片背景色
	var backgroundColor by remember { mutableStateOf(0) }
    val context = LocalContext.current
    //ImageRequest
    val modelBuilder = ImageRequest.Builder(context).data(item.url.getPicUrl()).crossfade(false)
        .allowHardware(false).transformations().placeholder(R.drawable.ic_pokeball)
        .error(R.drawable.ic_pokeball)

	//开启一个获取协程的附带效应当ImageRequest发生改变时，通过ImageRequest来动态获取bitmap
	//再通过Palette来获取其主色
    LaunchedEffect(modelBuilder.build()) {
        val bitmap = context.imageLoader.execute(modelBuilder.build()).drawable?.toBitmap(
            config = Bitmap.Config.RGBA_F16
        )
        bitmap?.let {
            val palette = Palette.from(bitmap).generate()
            //当backgroundColor得到值，可组合项重组
            backgroundColor = palette.getDominantColor(0)
        }
    }
```

上面的代码最终能实现我想要的效果，但是我始终觉得不够优雅，后面有机会在封装一下吧

## 一个进度缓慢增加的圆形进度条
也是封装好的通用组件**CommonCircularProgress.kt** 

```kotlin
@Composable
fun CommonAttributeCircularProgress(text:String,content:String,mProgress: Float,modifier: Modifier){
    Column(
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        var progress = remember {
            mutableStateOf(0.0f)
        }
        LaunchedEffect(true){
            var state = 0.0f
            while (state <= mProgress) {
                progress.value = state
                state += mProgress / 10f
                delay(50)
            }
        }

        Box(){
            Text(text = text,modifier = modifier.align(Alignment.Center))
            CircularProgressIndicator(progress = 1f,
                color = Color(0xFFffcba4),
                modifier = modifier
                    .align(Alignment.Center)
                    .size(150.cdp, 150.cdp)
            )
            CircularProgressIndicator(progress = progress.value,
                color = MaterialTheme.colors.secondary,
                modifier = modifier
                    .align(Alignment.Center)
                    .size(150.cdp, 150.cdp)
            )
        }
        Text(text = content, modifier = modifier.padding(top = 10.cdp))
    }
}
```
我没有找到能给CircularProgressIndicator可组合项目设置进度条范围内的背景的属性，所以使用了box将其覆盖一个CircularProgressIndicator可组合项来实现其背景的假象
## 单Activity使用navigation跳转Compose可组合项返回时页面重组的问题

列表项数据我按照官方推荐使用的方法直接使用了**collectAsLazyPagingItems**，然后我发现它确实起作用了，但是也发生了上诉问题，我每一次从详情页返回到列表页，列表页都会闪烁然后重组。这是因为每一次返回，可组合项都重新进行了请求导致数据发生变化，从而导致重组，为了解决这一问题，我在viewmodel中保存了一份请求得到的数据

```kotlin
@HiltViewModel
class PokemonListViewModel @Inject constructor(
    private val pokemonRepository: PokemonRepository
) :
    ViewModel() {


    // 宝可梦列表数据流  currentResult是为了保证从详情页面返回时不重新加载数据
    var currentResult: Flow<PagingData<PokemonResult>>? = null

    fun getPokemon(searchString: String?): Flow<PagingData<PokemonResult>> {
        val newResult = pokemonRepository.getPokemon(searchString).cachedIn(viewModelScope)
        currentResult = newResult
        return newResult
    }



}
```
并在页面中这样正确使用**collectAsLazyPagingItems**

```kotlin
    var list: LazyPagingItems<PokemonResult>? = null
    list = if (vm.currentResult != null) {
        vm.currentResult!!.collectAsLazyPagingItems()
    } else {
        vm.getPokemon(searchString.value).collectAsLazyPagingItems()
    }
```

这样当我从详情页返回到列表页时，页面显示正常

##  hiltViewModel()

在使用viemodel时，一开始我按照官方使用viewmodel的示例使用了**viewModel()**方法，并且在调试中是正常的。

```kotlin
val vm: PokemonListViewModel = viewModel()
```

可当我在加入**navigation** 后，viewmodel并不能正常的工作，后来查阅资料发现要使用

```kotlin
    implementation "androidx.navigation:navigation-compose:2.5.0-beta01"
```

包中的 **hiltViewModel()**

```kotlin
val vm: PokemonListViewModel = hiltViewModel()
```

---

# 主要参考项目

Jetpack Compose仿写网易云音乐
[NCMusic](https://github.com/sskEvan/NCMusic);  
一个使用原生xml kotlin写的宝可梦图鉴
[PokeApi-Pokedex](https://github.com/ronnieotieno/PokeApi-Pokedex);

# 总结
整个项目其实就两个可组合项，一个是FullScreenView（列表页），一个是AttributeDetailView（详情页），每个页面就200行代码左右，对比起原生xml，的确是少了很多样板代码

