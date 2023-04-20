---
title: kotlin 将List＜Any＞转为List＜Object ＞
date: 2023-01-16 16:20:08
tags: kotlin
categories: andorid
cover: "/images/android2.jpg"
---

```kotlin
import com.google.gson.Gson; 

inline fun <reified T> List<Any>.toBeanList(): List<T> {
        val newList : MutableList<T> = arrayListOf()
        for (item in this){
            val toJson = gson.toJson( item)
            val entity: T = gson.fromJson(toJson,T::class.java)
            newList.add(entity)
        }
        return newList.toList()
    }
```

主要是记录inline与reified，怕忘记了，另一个就是书不要读太死了，灵活变通吧
