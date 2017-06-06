---
title: Dagger注入框架入门概念理解
date: 2017-06-06 10:08:50
tags: [Android]
categories: [Android, dagger]
---

# 为什么要使用这个框架
目前主流的Android应用架构主要是MVP，能够在MVC的基础上解放View层的代码，但是其实大部分的代码还是转移到了P层，在P层处理了大部分的V层的显示逻辑。P层代码还是稍显臃肿。
下面给出大概的MVP示例代码

```java
//View   A.java
class A extends View implements IHello{
    MyPresenter presenter;

    public A (Context context){
        super(context);
        presenter = new MyPresenter(this);
        presenter.doSomethingOutOfMainThread();
    }

    @Override
    public void showHello(){
        Log.i("a", "Hello from A");
    }
}

//View层暴露给P层的接口 IHello.java
public interface IHello {
    void showHello();
} 

//P层  MyPresenter.java

class MyPresenter {

    private WeakReference<IHello> mHello;

    public MyPresenter(IHello view){
        this.mHello = new WeakReference<>(view);
    }

    public void doSomethingOutOfMainThread(){
        /**
         * 执行了一些耗时的操作，以及一大堆的数据逻辑处理操作之后
         */
        if (this.mHello != null){
            this.mHello.get().showHello();
        }


    }

}


```


上面的代码比较简单，但是有一个地方，可以看得出来，代码非常的粗糙，以及机械化。
```java
    public A (Context context){
        super(context);
        //这句
        presenter = new MyPresenter(this);
    }
```

如果这个界面非常的复杂，需要引入很多模块下的P层代码怎么办？难道这样做？
```java
    public A (Context context){
        super(context);
        presenter1 = new MyPresenter1(this);
        presenter2 = new MyPresenter2(this);
        presenter3 = new MyPresenter3(this);
        presenter4 = new MyPresenter4(this);
        presenter5 = new MyPresenter5(this);
        presenter6 = new MyPresenter6(this);
    }
```
上面的数字只是简化，不是重点，重点应该发现了，如果界面功能复杂，那么很有可能代码就会变成这样，非常的丑陋。即便吧这部分工厂的代码抽离，那么工厂类也是这样非常的丑陋。

<!-- more -->

# Dagger的基本概念，极简版

**吐槽一下**
在网上查了一些资料后，发现很心寒。大部分的文章不是互相转载，要么就是长篇大论，裹脚布一样。一些文章打着标题“快速入门”，结果进去文章少说有上万字。看半天发现人看晕了。十分无语，这里我要吐槽一下，你写这篇文章的初衷是什么？显示自己的文采非常好？还是为了方便自己理解？还是为了方便帮助别人快速的理解？还是为了简单的让别人觉得你文章写的越长，技术就越牛逼？

**开始**
基本流程：  Activity->Component->Module->Privide 
接着上示例代码：
```java

//第一步
//module层 MyModule.java
@Module
public class MyModule {
    private IHello mIHello;
    public MyModule(IHello view){
        this.mIHello = view;
    }

    @PerActivity
    @Provides
    public MyPresenter providerMyPresenter(){
        //这里是最终提供View需要元素的工厂方法的具体实现的地方
        //这里的代码会在View调用注入的过程中调用，View层需要inject什么对象，就调用返回什么对象的方法。
        return new MyPresenter(this.mIHello);
    }
}

//第二步 
//Component层  MyComponent.java
@PerActivity
@Component(modules = MyModule.class)
public interface MyComponent {
    void inject(MyActivity activity);
}

//第三部，执行Android studio Build->rebuild 让Dagger生成一些文件

//第四部
// View层  MyActivity.java
public class MyActivity extends Activity implements IHello{
    @Inject
    private MyPresenter presenter;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState){
        DaggerMyComponent.builder()     //DaggerXXX都是Dagger框架生成的
            .myModule(new MyModule(this))
            .build()
            .inject(this); //在这里，就能够通过Component层然后在往下Module层找到相应的注入对象的代码并执行。

        presenter.doSomethingOutOfMainThread();
    }

    @Override
    public void showHello(){
        Log.i("MyActivity", "Hello from MyActivity");
    } 
}

```


上面的@PerActivity 的注解的概念可以参考[@Scope](http://blog.csdn.net/afanyusong/article/details/52823416)

上面代码仅仅只是展现了Dagger使用的基本流程，但是上面的那种糟糕的引入代码要怎么解决？
有没有看见
``` java
//下面这句，注解的参数中modules 可以接受数组
@Component(modules = MyModule.class)  // @Component(modules = {MyModule1.class, MyModule2.class})
public interface MyComponent {
    void inject(MyActivity activity);
}
```

看见了吗？如果想要引入多个模块的P层，其实你不用一直new对象，仅仅只需要在V层注解要引入的P层对象，然后在Component中，引入需要的模块。这样就解决了上面的那种代码导致的极高的耦合性，方便了后期的维护。


# 结束

以上如果还有不明白的地方，可以一起讨论。仅仅是入门，其实这个框架仅仅是解决了P层的耦合，如果想要解决V层和M层的耦合，让V层不用import任何的M层的包以及类，还需要对P层进行更进一步的封装，以及对应用列表界面数据适配这块的改造。使得MVP的架构能够发挥最大的代码解耦的功效。这部分的思路以及代码会在后续的文章中写出。

个人联系方式：
QQ：781571323

