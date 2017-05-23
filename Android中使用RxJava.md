---
title: Android中使用RxJava
date: 2017-05-19 16:44:49
tags: [RxJava]
categories: [Android]
---

早期的代码由于没有考虑过度的控件添加带来的性能损耗，导致应用的体验非常的卡顿。
近期在项目重构的过程中，同事建议使用RxJava来重构应用，从原先的面向过程的变成方式，变为响应式编程。这个是有很大的好处的。

## 引入RxJava

``` gradle
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
``` java
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

``` java
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
				//对铺展开后的每一个数据进行处理
            }
        });
```

## 线程切换

上面的例子中都可以看到**".subscribeOn"** 和 **".observeOn"**这两个方法，第一次了解的时候，不明白，但是多用几次就能很明显的知道，subscribeOn这个是定义耗时操作的线程，observeOn是定义回调通知的线程。

通过了解上面的几个API，就能用RxJava适应大部分的开发需求。

## 延伸扩展 RxBus

这里的控件之间交互通信的框架有很多，类似EventBus,otto。但是为了适配RxJava的代码，就用了这个框架，里面的发送事件的原理和android的Handler差不多。
来点示例：
``` java
public class RxBus {

//    private static volatile RxBus mInstance;
    private SerializedSubject<Object, Object> mSubject;
    private HashMap<String, CompositeSubscription> mSubscriptionMap;

    public RxBus() {
        mSubject = new SerializedSubject<>(PublishSubject.create());
    }

    // 这里使用Dagger来处理单例
    /*
    private RxBus() {
        mSubject = new SerializedSubject<>(PublishSubject.create());
    }

    public static RxBus getInstance() {
        if (mInstance == null) {
            synchronized (RxBus.class) {
                if (mInstance == null) {
                    mInstance = new RxBus();
                }
            }
        }
        return mInstance;
    }*/

    /**
     * 发送事件
     *
     * @param o
     */
    public void post(Object o) {
        mSubject.onNext(o);
    }

    /**
     * 返回指定类型的Observable实例
     *
     * @param type
     * @param <T>
     * @return
     */
    public <T> Observable<T> toObservable(final Class<T> type) {
        return mSubject.ofType(type);
    }

    /**
     * 是否已有观察者订阅
     *
     * @return
     */
    public boolean hasObservers() {
        return mSubject.hasObservers();
    }

    /**
     * 一个默认的订阅方法
     *
     * @param type
     * @param next
     * @param error
     * @param <T>
     * @return
     */
    public <T> Subscription doSubscribe(Class<T> type, Action1<T> next, Action1<Throwable> error) {
        return toObservable(type)
                // 加上背压处理，不然有些地方会有异常，关于背压参考这里：https://gold.xitu.io/post/582d413c8ac24700619cceed
                .onBackpressureBuffer()
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(next, error);
    }

    /**
     * 保存订阅后的subscription
     * @param o
     * @param subscription
     */
    public void addSubscription(Object o, Subscription subscription) {
        if (mSubscriptionMap == null) {
            mSubscriptionMap = new HashMap<>();
        }
        String key = o.getClass().getName();
        if (mSubscriptionMap.get(key) != null) {
            mSubscriptionMap.get(key).add(subscription);
        } else {
            CompositeSubscription compositeSubscription = new CompositeSubscription();
            compositeSubscription.add(subscription);
            mSubscriptionMap.put(key, compositeSubscription);
        }
    }

    /**
     * 取消订阅
     * @param o
     */
    public void unSubscribe(Object o) {
        if (mSubscriptionMap == null) {
            return;
        }

        String key = o.getClass().getName();
        if (!mSubscriptionMap.containsKey(key)){
            return;
        }
        if (mSubscriptionMap.get(key) != null) {
            mSubscriptionMap.get(key).unsubscribe();
        }

        mSubscriptionMap.remove(key);
    }
}
```

这个是从一个开源项目中拷贝出来的代码，其中有些不合理的代码设计，但是不影响主体的实现事件控件之间交互的功能。可以用来入门学习。

以上是针对RxJava的入门学习，但是这个仅仅只是很小的一部分，剩余的还有很多，例如和Retrofit框架的搭配使用，还有在MVP设计思路中的实现，这个都需要很多的实践，也欢迎有志同道合的朋友能够一起学习互相联系。

联系方式： 
QQ:781571323
wechat:18559176792

