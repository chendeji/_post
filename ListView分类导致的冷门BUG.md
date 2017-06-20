---
title: ListView分类导致的冷门BUG
date: 2015-06-01 11:44:08
tags: [Bug]
categories: [Android]
---

## ListView分类显示控件中遇到的冷门BUG
这个BUG非常的冷门，基本都是用不到的。

**需求**
在一个列表控件中显示多种类型的控件，根据数据给出的分类不同来显示。目前开发的项目中，购物车列表中，需要显示不同类型的购物项数据。

目前采用的技术解决方案是用ExpandableListView 嵌套 ListView，ListView需要特殊的处理，不然会有滑动控件嵌套滑动控件内容显示不完整的问题。

代码如下：
```java 
public class NoScrollListView extends ListView {
    public NoScrollListView(Context context) {
        super(context);
    }
    public NoScrollListView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec){
        int expandSpec = MeasureSpec.makeMeasureSpec(Integer.Max_Value >> 2, MeasureSpec.AT_MOST);
        super.onMeasure(widthMeasureSpec, expandSpec);
    }
}
```

上面的代码可以很容易的看出，是要修改布局ListView控件的时候要显示的高度大小。

## 设置数据适配器
ListView 设置数据适配器的时候，假如要做分类显示不同控件的功能，需要注意的是两个回调方法：
```java
@Override
public int getChildTypeCount(){
    return 2; //错误，应该返回4
}

@Override
public int getChildType(int groupPostion, int childPosition){
    return 3;
}
```

上面的这两个回调方法，返回的数据要注意一个规则，就是getChildType > getchildTypeCount 不然就会爆出OutOfBounts的数组越界的错误。