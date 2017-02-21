---
title: Android 自定义View (一)
date: 2017-02-21 11:42:13
tags: Android, View
categories: Android
---
# Android 自定义View（一）

##概述
Android框架中创建了许多可供开发者调用的view，这些view能够与用户交互并且展示多样的数据。但是，当你的应用更加独特复杂的时候，内部创建的view可能已经无法满足你的需要。这时候就需要自己创建所需的view了。

**那么如何创建自定义view呢? 哈哈， 看官别急，请听我慢慢道来。**

## View绘制流程
<img src="https://camo.githubusercontent.com/2fff630a00a9a8a64a227601235984c6cb3b0fcb/687474703a2f2f7777342e73696e61696d672e636e2f6c617267652f30303558746469326a7731663633387772657537346a333066633068656161792e6a7067"/>

<!-- more -->
Android中View 的绘制过程包括三个最重要的方法，即onMeasure、onLayout和onDraw方法。在此不着重讲。
##自定义View流程
* 自定义属性
* 扩展View 类或者扩展 View的某子类，例如Button
* 在自定义View中获取自定义的属性
* 重写onDraw()方法

### 自定义属性
Android中内建的View都是可以通过XML元素控制展现的，所以一个良好的自定义View也要能通过XML构建。
* 在`res/value/`目录下创建attrs.xml文件
* 首先在`<declare-styleable>`资源元素中定义自定义的属性
* 在XMl文件中指定元素的值
* 运行中获取元素值
* 在自定义View中应用获取的元素值

``` xml
<resources>
    <declare-styleable name="AlarmClockView">
        <attr name="ACVCircle_radius" format="dimension" />
        <attr name="ACVCircle_color" format="color" />
        <attr name="ACVCircle_change_color" format="color" />
    </declare-styleable>
</resources>
```
本文件定义了圆的半径，颜色以及更改颜色三个属性，format是属性的取值类型， 这三个属性属于AlarmClockView类型实体，也就是说一般用该名字作为扩展View类的类名。

 当我们定义了自定义属性后，就要在布局文件中应用他们了。应用时，自定义属性与内建属性没区别，唯一的不同之处在于不同的命名空间。自定义属性的命名空间时`http://schemas.android.com/apk/res/[your package name]`

``` xml
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">
        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
            <com.example.youmiclock.AlarmClockView
                android:id="@+id/alarm_clock_view"
                android:layout_width="300dp"
                android:layout_height="300dp"
                android:layout_centerHorizontal="true"
                app:ACVCircle_radius="50dp"
                app:ACVCircle_color="#FFD6D6D6"
                app:ACVCircle_change_color="#26a69a" />
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="中国标准时间"
                android:layout_centerHorizontal="true"
                android:layout_below="@id/alarm_clock_view"
                android:layout_marginTop="-30dp"/>
        </RelativeLayout>
        <android.support.v7.widget.RecyclerView
            android:id="@+id/recycler_view"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
        </android.support.v7.widget.RecyclerView>
    </LinearLayout>
</android.support.design.widget.CoordinatorLayout>
```
**这里明明空间用的`xmlns:app="http://schemas.android.com/apk/res-auto"`是因为在Gradle工程中，工具会找到最正确的命名空间，不要硬编码自己的命名空间**

###应用自定义属性
创建一个类继承于`View`或其子类，然后在构造方法中获取自定义属性.官方文档写到，虽然XML标签被传递到了`AttributeSet` 对象中，我们可以从中取得其值，但是这种方式有缺陷，所以我们要将`AttributeSet`传递到`obtainStyledAttributes()`中，该方法返回一个`TypeArray`对象。
``` java
    public AlarmClockView(Context context) {
        super(context);
        initPaint();
    }

    public AlarmClockView(Context context, AttributeSet attrs) {
        super(context, attrs);
        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.AlarmClockView);

        try {
            circleRadius = typedArray.getDimensionPixelSize(R.styleable.AlarmClockView_ACVCircle_radius, 50);
            circleColor = typedArray.getColor(R.styleable.AlarmClockView_ACVCircle_color, Color.parseColor("#cfd8dc"));
            circleChangeColor = typedArray.getColor(R.styleable.AlarmClockView_ACVCircle_change_color, Color.BLACK);
        }finally {
            typedArray.recycle();
        }
        initPaint();
    }
```
**记得回收TypeArray对象**

View的构造函数有四中重载方式，

```java
  public void AlarmClockView(Context context) {}
  public void AlarmClockView(Context context, AttributeSet attrs) {}
  public void AlarmClockView(Context context, AttributeSet attrs, int defStyleAttr) {}
  public void AlarmClockView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {}
```
四个参数的狗仔函数在API21时才添加，暂时不考虑
三个参数的第三个参数一般不用，也暂时不考虑
故只剩下一个参数和两个参数的构造函数
以下方法调用的是一个参数的构造函数：

``` java
  //在Avtivity中
  AlarmClockView view = new AlarmClockView(this);
```
以下方法调用的是两个参数的构造函数：

``` xml
  //在layout文件中 - 格式为： 包名.View名
            <com.example.youmiclock.AlarmClockView
                android:id="@+id/alarm_clock_view"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_centerHorizontal="true"
                app:ACVCircle_radius="100dp"
                app:ACVCircle_color="#FFD6D6D6"
                app:ACVCircle_change_color="#26a69a"
                android:background="#000000"/>
```
现在的效果是， 黑色背景是为了突出尺寸
![Alt text](./Screenshot from 2017-02-21 10-00-47.png)

### onMeasure()

**那么问题来了，我们在xml文件中已经指定了宽高，为什么还要在自定义View中再次测量设置么？**
因为View的大小不仅由自身决定，同事也会收到父控件的影响，为了能更好的适应各种情况，一般需要自己测量。

**MeasureSpec共有三种测量模式**
| 模式名称 |  二进制数值 | 描述 |
| :-----------| ---------------:| :-----:|
| UNSPECIFIED | 00 |默认值，父控件没有给子view任何限制，子View可以设置为任意大小|
| EXACTLY | 01 | 表示父控件已经确切的指定了子View的大小|
| AT_MOST | 10 | 表示子View具体大小没有尺寸限制，但是存在上限，上限一般为父View大小|

**如果对View的宽高进行修改了，不要调用super.onMeasure(widthMeasureSpec,heightMeasureSpec);要调用setMeasuredDimension(widthsize,heightsize); 这个函数。**

``` java
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {

        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);

        int heightMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSize = MeasureSpec.getSize(heightMeasureSpec);

        int width;
        int height;

        if(widthMode == MeasureSpec.EXACTLY){
            width = widthSize;
        }else {
            width = widthSize / 2;
            circleRadius = 80;
            oval.set(-80, -80, 80, 80);
        }

        if(heightMode == MeasureSpec.EXACTLY){
            height = heightSize;
        }else {
            height = heightSize / 2;
        }

        setMeasuredDimension(width, height);
    }
```
现在的效果是， 可能不太美观，有需要的自己再调一下合适的尺寸。
![Alt text](./Screenshot from 2017-02-21 10-00-23.png)

**按照流程来讲，下面就应该是布局咯，即onLayout， 本段内容后续补齐吧**

### onDraw
到了最重要的部分咯，即绘制部分，使用的是Canvas绘图

```java
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        Log.d(TAG, "onDraw: " + hour + ":" + minute + ":" + second);
        canvas.translate(getWidth()/ 2, getHeight()/ 2);
        canvas.drawCircle(0, 0, circleRadius, circlePaint);
        getDegree();
        // canvas.drawRect(oval, radiusPaint);
        canvas.drawArc(oval, -90, secondDegree, false, radiusPaint);
    }
```
效果图， 绿色线条会随着时间增加，一秒6度。看来我还是要先高搞清楚怎么制作动态图片。Crying。
![Alt text](./Screenshot from 2017-02-21 11-03-34.png)

## 总结
好啦，本篇暂时记录到这里，关于后续的自定义View，我会继续学习，继续记录，加油。