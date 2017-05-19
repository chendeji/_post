---
title: Android中使用RxJava
date: 2017-05-19 16:44:49
tags: [RxJava]
categories: [Android]
---

早期的代码由于没有考虑过度的控件添加带来的性能损耗，导致应用的体验非常的卡顿。
近期在项目重构的过程中，同事建议使用RxJava来重构应用，从原先的面向过程的变成方式，变为响应式编程。这个是有很大的好处的。

##引入RxJava
```
//rxjava
compile 'io.reactivex:rxjava:1.1.9'
compile 'io.reactivex:rxandroid:1.2.1'
compile 'com.jakewharton.rxbinding:rxbinding:0.4.0'
compile 'com.trello:rxlifecycle:1.0'//关于生命周期的控制，防止内存泄漏
compile 'com.trello:rxlifecycle-components:1.0'
compile'com.tbruyelle.rxpermissions:rxpermissions:0.9.1@aar'
```


