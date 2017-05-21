---
title: Android中使用RxJava
date: 2017-05-19 16:44:49
tags: [RxJava]
categories: [Android]
---

早期的代码由于没有考虑过度的控件添加带来的性能损耗，导致应用的体验非常的卡顿。
近期在项目重构的过程中，同事建议使用RxJava来重构应用，从原先的面向过程的变成方式，变为响应式编程。这个是有很大的好处的。

## 引入RxJava
```
//rxjava
compile 'io.reactivex:rxjava:1.1.9'
compile 'io.reactivex:rxandroid:1.2.1'
compile 'com.jakewharton.rxbinding:rxbinding:0.4.0'
compile 'com.trello:rxlifecycle:1.0'//关于生命周期的控制，防止内存泄漏
compile 'com.trello:rxlifecycle-components:1.0'
compile'com.tbruyelle.rxpermissions:rxpermissions:0.9.1@aar'
```

<!-- more -->

## RxJava的优势
1. **线程的调度**，这个优势，会让你的线程切换代码减少很多，最大的优点，也是我最喜欢的。

2. **流式代码**，说这个太抽象，直接上示例。
```
        Observable.create(new Observable.OnSubscribe<Object>() {
            @Override
            public void call(Subscriber<? super Object> subscriber) {

            }
        })
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Action1<Object>() {
            @Override
            public void call(Object o) {

            }
        });
```
代码是不是很美观？所有的调度，都在"."方法调用后完成，对于我这样有点代码洁癖的，除非是项目非常的紧急，没太多时间思考，代码确实也是会写的很乱。但是有了这个，确实就不用花太多的脑力在代码美观上。 而且还兼具了性能上的优化，上面两点是我的最爱。

3. 还有一个比较大的优势，就是对嵌套机构的优化，这点其实可以归并到第二点，但是个人感觉有必要分出来。**多维铺展** 这个是我个人的总结而已，简单的来讲，就是可以对多维的数据进行展开，中间可以做很多的操作，例如过滤不正常的数据，或者将数据进行转换。在这里也是上点例子。

```
        Observable.just(isRefresh)
                .filter(new Func1<Boolean, Boolean>() {
            @Override
            public Boolean call(Boolean aBoolean) {
                //对不符合条件的内容进行过滤
                return null;
            }
        }).map(new Func1<Boolean, Object>() {
            @Override
            public Object call(Boolean aBoolean) {
                //对筛选出来的数据进行转换
                return null;
            }
        }).flatMap(new Func1<Object, Observable<String>>() {
            @Override
            public Observable<String> call(Object o) {
                //对转换后的数据，进行多维铺展
                return null;
            }
        })
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Action1<String>() {
            @Override
            public void call(String s) {

            }
        });
```



