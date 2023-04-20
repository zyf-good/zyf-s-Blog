---
title: Android在MVVM中使用适配器模式的一些思考
date: 2022-12-23 17:02:41
tags: 设计模式
categories: andorid
cover: "/images/andorid1.jpg"
---


---

# 前言
人总是在不断地学习中进步，在学习了设计模式过后，一直想将其用到项目当中，将其融会贯通，但是强行使用反而会适得其反，因此，还是需要在合适的地方使用，方为正道。*本文将会介绍在一个fragment中通过适配器设计模式切换不同的viewmodel来展示不同的数据，以达到替换viewpager+fragment的效果。*

---

# 一、适配器设计模式是什么？

如果我们要在手机屏幕上展示如下列表，学过Andorid的朋友都知道，展示一个列表需要使用recycleview，然后新建一个布局展示item，然后就是我们今天的重头戏，**adapter（适配器 ）**，我们需要创建一个adapter来讲item布局填充到recycleview中并展示出来。

这时有的朋友就要笑了，你不会跟我说这就是适配器模式吧？那我会了；没错，这就是适配器模式，通过切换不同的adapter，你可以展示不同的item数据，这其中recycleview是被适配者，我们创建的adapter是具体的适配给recycleview的实现，而我们继承的 RecyclerView.Adapter 
则是一个通用的规范，告诉recycleview我们是来适配的。

如果用更通俗易懂的话来说，就相当于你去找工作，然后跟老板说我要面试安卓开发工程师，
老板就是被适配者，你就是具体的适配，安卓开发工程师就是约定的规范




# 二、MVVM中使用适配器模式
## 1.原因

> 再先说一下哦，不要强行使用设计模式，因为那会显得你在炫技。

接到一个需求是要在一个fragment中放一张高德地图，然后在地图中动态切换tab过后，切换展示在地图上的数据，本着能抄绝不自己写的工作态度，我在需求会还没开的时候，就在看隔壁业务组的这个需求是怎么写的了，然后我就感受到了隔壁业务组对这个世界深深的恶意，先说下他们的实现吧，屏幕上方是一个tablayout，下方是个viewpager，带了6个模块的fragment，然后每一个fragment都是相同的地图逻辑和布局，唯一不同的是展示数据不同，筛选项不同，每一个fragment达到了惊人的3000+行的代码量，通篇都是重复的逻辑和令人昏昏欲睡的代码，我看了半个小时就困的不行了，醒来过后长叹息以掩泣兮，还是自己写吧；在这个需求的背景下使用这个写法，我只能说脑袋被门夹了。于是就想到了使用适配器模式。毕竟，真的只有展示的数据和筛选项不同，UI都是大同小异的，能用一个布局为什么要用六个呢，是吧


## 2.实现
敲定了使用的方法过后，就是设计的阶段了，因为项目是MVVM的，最终敲定对viewmodel下手，因为viewmodel就是对数据的处理，而我们正好差异就在数据层，因此我新创建了一个实体类

```kotlin
class MapEntity {
    //区域编号
    var areaCode = ""
    //区域名称
    var areaNames = ""
    //纬度
    var latitude = ""
    //经度
    var longitude = ""
    //统计数量
    var mapCount = 0

    var dataList : List<Any> = emptyList()

}
```
这里的dataList就是我准备用来展示的数据，但是因为每一个数据类型都有差异，所以没有声明具体的数据类型，而是使用Any代替。
然后定义一个安卓开发工程师的职位；（新建一个viewmodel的接口），类似如下

```kotlin
interface IBaseMapViewModel {

    var getList: MutableLiveData<List<Any>>
        get() = MutableLiveData<List<Any>>()
        set(value) = TODO()

    var isEmpty: MutableLiveData<Boolean>
        get() = MutableLiveData()
        set(value) = TODO()
    var loading: MutableLiveData<Boolean>
        get() = MutableLiveData()
        set(value) = TODO()
        
    var hintString : String

    var type : Int

    var drawable : Int

    var drawableClick : Int

    fun getMapData(
        botRightLat: String,
        botRightLon: String,
        categoryNo: String,
        industryNo: String,
        thirdCategoryNo: String,
        keyWord: String,
        searchType: Int,
        topLeftLat: String,
        topLeftLon: String,
        context: Context,
        distance: String,
        mCenterLatLng: LatLng,
        employmentStatus: String,//用工状态
        proficiency: String, //专业职级 	熟练度（专业职级 1:高级工(大工) 2:中级工(中工) 3:初级工(小工)）
        workYear: String, //工龄 1-3传1 3-5传2 5-10传3 10以上传4
        qualificationTypeNo: String,
        qualificationClassNo: String,
        teamNumber: String,
        businessType : String
    )

    //成都房建水泥工成都房建水… 2.2w/月
    fun getMarkerText(item: MapEntity):String

    //等12份工作
    fun getMarkerNum(item: MapEntity,isProvince:Boolean):String

    fun getUrlSetting( id:String,  type:Int,  modelType:String, userNo:String, context: Context)

}
```
可以看到，在这个抽象的viemodel中我们的入参包含了MapEntity；再看getMapData方法的入参可能会有点多，这是因为他包含了所有六个页面的筛选项，这也算是适配器模式的一种缺点吧

然后就是创建“你”这样一个viemodel啦

```kotlin
class MapViewModel () : BaseViewModel(), IBaseMapViewModel {
```
实现IBaseMapViewModel中的接口，并具体的处理每个数据实体类的展示，这样的具体业务viewmodel，我一共创建了六个！

最后怎么使用这些具体的viewmodel呢，在fragment中，先定义抽象的viewmodel

```kotlin
    //地图viewmodel
    private lateinit var viewModel: IBaseMapViewModel
```

然后通过tablayout点击不同的position来初始化不同的viewmodel

```kotlin
 when (selectSecondTap) {
            xxxxType -> {
                viewModel = MapViewModel()
            }
           
           ......
```

这里有个坑就是observe在每次重新初始化viewmodel后也要重新初始化，不然就会得不到请求后的数据


---

# 这样使用适配器模式的优缺点的总结
优点：
 - 减少了代码量，缩减了fragment的代码量，从3000+行缩减到了1000+行
 - 代码更清晰，基本上只有切换viewmodel的时候才有判断，其他都是实打实的业务代码
 - 减少维护成本，如果要在六个的基础上再新增数据展示只需要再加一个viewmodel和 逻辑判断，非常方便
 
 缺点：
 - 不同的地方要做额外的处理，如果不写注释，那一块代码看着就比较吓人了（例如getMapData的入参）
 - 如果需求出现巨大变更，例如不同的数据要采用不同的表现方式，就要推倒重写或者新增很多无意义的接口，让可读性和可维护性变得很差
 

