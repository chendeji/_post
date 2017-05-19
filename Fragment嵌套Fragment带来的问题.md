---
title: Fragment嵌套Fragment带来的问题
date: 2017-05-19 15:46:19
tags: [bug, Fragment]
categories: [Android]
---

## 问题原因
目前在做的项目中，在主页面MainActivity中是含有三个Fragment来做三个页面片段了，但是新增的需求中要求Fragment中要签到一个Viewpager小布局，然后Viewpager中又使用到了Fragment。那么这里就出现了问题。

```
MainActivity->Fragment->ViewPager->Fragment
```

之前在官方网站中有印象也有提示，是说尽量避免Fragment嵌套Fragment。有这方面的限制的原因，是Fragment管理器对象限制的问题。[getSupportFragmentManager](https://developer.android.google.cn/reference/android/support/v4/app/FragmentActivity.html#getSupportFragmentManager())获取到的管理器在这次的项目中仅仅只能支持到二级的Fragment，如果二级的Fragment容器有两个以上，那么很遗憾，第二个之后的Fragment容器将不会显示任何的Fragment内容。

## 解决办法
在项目开发中应尽量避免使用Fragment嵌套，Fragment的使用确实有一定的好处，例如可以快速的回收内存资源。但是如果存在嵌套的情况，建议还是在Fragment下一级的界面里都是用普通的View控件。