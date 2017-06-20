---
title: Eclipes配置JNI环境相关
date: 2015-07-23 11:06:44
tags: [JNI]
categories: [Android, JNI]
---

# JNI在Eclipes中常用的配置设置
在有使用JNI来开发的项目，需要让项目支持底层代码开发，具体操作如图
右键点击工程 Android tools -> Add Native Surpport 添加 native support
![支持底层代码](http://img.blog.csdn.net/20150913215746047)
然后接着按Finish结束操作

<!-- more -->

接着将NDK中的头文件导入到项目中，支持项目中C/C++代码的引用
![](http://img.blog.csdn.net/20150913220306307)
![](http://img.blog.csdn.net/20150913220325399)
![](http://img.blog.csdn.net/20150913220657164)
到了这里，就要指定一下NDK目录中的头文件目录的位置了
点击File system选中ndk 头文件所在目录 例如：${ANDROID_NDK_HOME}\platforms\android-21\arch-arm\usr\include
点击确定，配置完成

然后你输入就有提示了，但是要注意C++和C的文件编码风格有点不同，主要是因为C和C++内部的调用机制不同吧，细节的可能要去研究一下。

C++代码风格
```C++
#include "hello-jni.h"

/**
 * Class:     com_example_jnidemo_FirstJNIDemoActivity
 * Method:    stringFromJNI
 * Signature: ()Ljava/lang/CharSequence;
 */
JNIEXPORT jobject JNICALL Java_com_example_jnidemo_FirstJNIDemoActivity_stringFromJNI
  (JNIEnv *env, jobject obj){
    return env->NewStringUTF("Hello JNI");  //这里这句代码和C的代码调用不同
}
```


C代码风格
```C
#include "hello-jni.h"

/**
 * Class:     com_example_jnidemo_FirstJNIDemoActivity
 * Method:    stringFromJNI
 * Signature: ()Ljava/lang/CharSequence;
 */
JNIEXPORT jobject JNICALL Java_com_example_jnidemo_FirstJNIDemoActivity_stringFromJNI
  (JNIEnv *env, jobject obj){
    return (*env)->NewStringUTF(env, "Hello JNI");  //这里的代码和C++的调用方式就不同了
}
```


# 设置javah 生成头文件的命令工具

到这里基本的NDK的配置完成了。但是在jni开发中还有一些便捷的操作可以提供使用
在生成头文件的时候要一直去敲javah。非常的麻烦，这里eclipes可以添加这样的命令配置

Run -> External Tools -> External Tools Configuration 
左边栏中右击Program 增加 命令

Name : javah

Location : 
```
${system_path:javah}
```

Working Directory : 
```
${project_loc}/jni
```

Arguments : 
```
-classpath "${project_classpath};${env_var:ANDROID_SDK_HOME}/platforms/android-23/android.jar" ${java_type_name}
```

在Common标签页中，Display in favorites menu中，勾选External Tools 让你自定义编辑的javah命令能够显示在External Tools 的列表中。

在Refresh标签页中，点击单选 The project containing the selected resource。
这样调整完后，就可以了
