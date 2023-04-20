---
title: >-
  安卓使用fastjson解析带BigDecimal参数的实体类报错java.lang.NumberFormatException: For input
  string:乱码...
date: 2021-09-26 15:24:13
tags: 开发日常
categories: andorid
cover: "/images/android2.jpg"
---



# 发现问题

今天在公司工作的时用fastjson beancopy数据，出现了一个报错类似如下
```javascript
java.lang.NumberFormatException: For input string:
// 此处乱码
at java.lang.Integer.parseInt(Integer.java:521)
at java.lang.Integer.parseInt(Integer.java:556)
at java.math.BigDecimal.(BigDecimal.java:336)
at java.math.BigDecimal.(BigDecimal.java:374)
at com.alibaba.fastjson.parser.JSONLexerBase.scanFieldDecimal(JSONLexerBase.java:3727)
at com.alibaba.fastjson.parser.deserializer.JavaBeanDeserializer.deserialze(JavaBeanDeserializer.java:594)
at com.alibaba.fastjson.parser.deserializer.JavaBeanDeserializer.deserialze(JavaBeanDeserializer.java:295)
at com.alibaba.fastjson.parser.deserializer.JavaBeanDeserializer.deserialze(JavaBeanDeserializer.java:291)
at com.alibaba.fastjson.parser.deserializer.DefaultFieldDeserializer.parseField(DefaultFieldDeserializer.java:88)
at com.alibaba.fastjson.parser.deserializer.JavaBeanDeserializer.parseField(JavaBeanDeserializer.java:1282)
at com.alibaba.fastjson.parser.deserializer.JavaBeanDeserializer.deserialze(JavaBeanDeserializer.java:897)
at com.alibaba.fastjson.parser.deserializer.JavaBeanDeserializer.deserialze(JavaBeanDeserializer.java:295)
at com.alibaba.fastjson.parser.DefaultJSONParser.parseObject(DefaultJSONParser.java:706)

Caused by: com.alibaba.fastjson.JSONException: For input string:
//此处 乱码
at com.alibaba.fastjson.parser.DefaultJSONParser.parseObject(DefaultJSONParser.java:713)
at com.alibaba.fastjson.JSON.parseObject(JSON.java:394)
at com.alibaba.fastjson.JSON.parseObject(JSON.java:357)
at rxhttp.wrapper.converter.FastJsonConverter.convert(FastJsonConverter.java:59)
at rxhttp.wrapper.utils.Converter.convert(Converter.kt:45)
at rxhttp.wrapper.parse.AbstractParser.convert(AbstractParser.kt:33)
at com.galaxis.paser.ResultPaser.onParse(ResultPaser.java:41)
at rxhttp.wrapper.param.ObservableParser$SyncParserObserver.onNext(ObservableParser.java:103)
at rxhttp.wrapper.param.ObservableParser$SyncParserObserver.onNext(ObservableParser.java:54)
at rxhttp.wrapper.param.ObservableCallEnqueue$HttpDisposable.onResponse(ObservableCallEnqueue.java:79)
at okhttp3.internal.connection.RealCall$AsyncCall.run(RealCall.kt:504)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1133)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:607)
at java.lang.Thread.run(Thread.java:761)
```
我心中的当时十分平静，不就是不能强转吗？我来看看怎么强转不就行了，但是仔细一看不太对BigDecimal怎么转到Integer就不行了，也太low了吧
# 找到问题

然后就是漫长的查找问题的过程，发现报错的乱码和我代码里的都没有相符合的地方，最终把问题确定在了fastjson本身上，虽然有点不相信，但是排除法只能在它上面了。
功夫不负有心人，终于找到了github上面的这个issue，网址如下：
链接: [https://github.com/alibaba/fastjson/pull/3555](https://github.com/alibaba/fastjson/pull/3555).

大概说的就是修复了一个fastjson在Android系统上处理BigDecimal解析错误的一个bug，并且已经修复。

# 解决问题
我一看那好吧，谁叫我没本事不能自己解析呢是吧，既然你都修了我升个版本总可以用了吧，然后我发现最新的版本还是有这个问题，根本没提上去！！o(╥﹏╥)o

经历了九九八十一难，最后我使用了 fastjson自带的@JSONField的序列化和反序列化绕开了这个bug完成了解析

在BigDecimal的字段上加上了这个注解，然后自己写转换的方法，就可以绕开这个bug啦
```java
@JSONField(serializeUsing= BigDecimalToDouble.class,deserializeUsing = DoubleToBigDecimal.class)
```
BigDecimalToDouble .class
```java

import com.alibaba.fastjson.serializer.JSONSerializer;
import com.alibaba.fastjson.serializer.ObjectSerializer;

import java.io.IOException;
import java.lang.reflect.Type;
import java.math.BigDecimal;

/**
 * @ProjectName : ccpos
 * @Author : yifeng_zeng
 * @Time : 2021/9/23 12:51
 * @Description : fastjson轉換
 */
public  class BigDecimalToDouble implements ObjectSerializer {
    @Override
    public void write(JSONSerializer serializer, Object object, Object fieldName, Type fieldType, int features) throws IOException {
        BigDecimal value = (BigDecimal) object;
        Double doubleValue = value.doubleValue();
        serializer.write(doubleValue);

    }
}
```
DoubleToBigDecimal .class
```java
/**
 * @ProjectName : ccpos
 * @Author : yifeng_zeng
 * @Time : 2021/9/23 12:53
 * @Description : fastjson轉換
 */
public class DoubleToBigDecimal implements ObjectDeserializer {
    @Override
    public <T> T deserialze(DefaultJSONParser parser, Type type, Object fieldName) {
        Double value = parser.parseObject(Double.class);
        // DataType 数据类型 枚举
       return (T) new BigDecimal(value);


    }

    @Override
    public int getFastMatchToken() {
        return 0;
    }
}
```

还是挺简单的，不是吗？(简单个鬼，半天没有了，小声bibi）



