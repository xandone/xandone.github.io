---
layout: post
title: RxJava2 onNext called with null异常处理
date: 2021-03-08 15:12:32
tags: [编程]
toc:  true
---

最近维护项目的时候，遇到这个异常：  
Rxjava2后,当给onNext(T t)方法传递一个null对象的时候，会报异常"onNext called with null. Null values are generally not allowed in 2.x operators and sources"  

**分析：**  
一般后台给的数据格式是这样的：  
```json
"{"code":200,"message":"成功","data":{...}}"
```
data里面是我们需要的数据，正常情况下，即使data是个空数据，也应该给前台一个空的对象，而不应该是null，就像这次的bug，后台的数据是:  
```json
"{"code":200,"message":null,"data":null}"
```
针对于上面的bug，我觉得有以下几种解决方案：  

1.放在后端解决，联系后台改接口的数据格式，当空数据的时候，data修改成空数据并非是个null，或者code改成统一的某个非200码。  

2.前台解决，直接使用response作为返回值传递给onNext()函数，而不直接使用response.getData()，这样的response肯定不会是空，在具体的业务代码中对response.getData()实体类进行判断，以上可以避免以上异常。但是这样做有一个很明显的问题，无法统一处理code非200情况下的业务逻辑，项目中code非200情况下，通常会自定义一个ApiException统一处理，比如常见的“类型转换错误”，“证书验证失败”，“服务器连接失败”等等..  

3.前台解决，如果data为null，在给onNext()传值的时候进行非空判断，若为空，可以手动new一个空对象    
```java
(T) new TypeToken<T>(){}.getType()
```
进行传递。顺带说一下，如果返回类型是个List对象，可以直接返回Collections.emptyList()，这个方法返回的是Collections的一个final静态List对象，无法向里面写内容，更加安全且不占内存。  

4.前台解决，将方案2进行稍微的完善，使用flatMap操作符进行统一转化，但是不变换数据泛型的类型，只是为了处理统一处理code非200的情况，便于代码的后期维护，代码显得比较整洁。代码如下：
```java
public static <T> FlowableTransformer<BaseResponse<T>, BaseResponse<T>> handleBaseResponse() {
        return upstream -> upstream.flatMap((Function<BaseResponse<T>, Flowable<BaseResponse<T>>>) baseResponse -> {
            if (baseResponse.getCode() == 200) {
                return createData(baseResponse);
            } else {
                return Flowable.error(new ApiException(baseResponse.getMessage(), baseResponse.getCode()));
            }
        });
    }
```
**总结**  
但是我认为这种情况最好还是放在后台解决，普遍情况下，直接使用response.getData()实体数据比较方便，省去很多不必要的代码，这个异常的出现，归根在于后端同学代码的不完善。  