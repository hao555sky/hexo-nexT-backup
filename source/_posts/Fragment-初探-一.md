---
title: Fragment 初探(一)
date: 2017-04-20 15:33:10
tags: Android, Fragment
categories: Android
---
### 概述
最近写项目的时候, 用到了fragment, 就想着系统学习一下,实现完还要整理下,毕竟技术这种东西,稍微时间长点就会忘记.

### 设计原理
Android 在 Android 3.0（API 级别 11）中引入了片段，主要是为了给大屏幕（如平板电脑）上更加动态和灵活的 UI 设计提供支持。由于平板电脑的屏幕比手机屏幕大得多，因此可用于组合和交换 UI 组件的空间更大。利用片段实现此类设计时，您无需管理对视图层次结构的复杂更改。 通过将 Activity 布局分成片段，您可以在运行时修改 Activity 的外观，并在由 Activity 管理的返回栈中保留这些更改。

您应该将每个片段都设计为可重复使用的模块化 Activity 组件。也就是说，由于每个片段都会通过各自的生命周期回调来定义其自己的布局和行为，您可以将一个片段加入多个 Activity，因此，您应该采用可复用式设计，避免直接从某个片段直接操纵另一个片段。 这特别重要，因为模块化片段让您可以通过更改片段的组合方式来适应不同的屏幕尺寸。 在设计可同时支持平板电脑和手机的应用时，您可以在不同的布局配置中重复使用您的片段，以根据可用的屏幕空间优化用户体验。 例如，在手机上，如果不能在同一 Activity 内储存多个片段，可能必须利用单独片段来实现单窗格 UI。
![Alt text](/assets/images/fragment/fragmentsInPhoneAndTablet.png)

  例如 — 仍然以新闻应用为例 — 在平板电脑尺寸的设备上运行时，该应用可以在 Activity A 中嵌入两个片段。 不过，在手机尺寸的屏幕上，没有足以储存两个片段的空间，因此Activity A 只包括用于显示文章列表的片段，当用户选择文章时，它会启动Activity B，其中包括用于阅读文章的第二个片段。 因此，应用可通过重复使用不同组合的片段来同时支持平板电脑和手机，如图 1 所示。

<!-- more -->

### Fragment生命周期

#### Fragment生命周期和其与Activity比较
![Alt text](/assets/images/fragment/fragment_lifecycle.png)                                                                    ![Alt text](/assets/images/fragment/activity_fragment_lifecycle.png)
`左边是fragment的生命周期, 右边是于Activity的比较`


#### Fragment的生命周期方法
> * onAttach() : fragment于activity关联时调用
> * onCreate() : 系统创建Fragment所调用的方法, 注意该方法调用的时候,activity还在创建中.因此,activity中的相关视图结构还未被创建完成.如果想使用activity中的某些资源,要在onActivityCreated中获取.
> * onCreateView() : 系统会在首次绘制fragment用户界面时调用此方法, 期望返回的view是fragment的布局视图, 也可以返回null


> 绘制方法是
> ``` java
public View onCreateView (LayoutInflater inflater,  ViewGroup container, Bundle savedInstanceState){
    View view = inflate.inflate(R.layout.fragment1, container, false);
	return view;
}
```
> 关于inflater方法我还有很多不理解的地方,以后会继续学习整理
> * onActivityCreated() : 表明activity已经完成了自身的onCreate()方法, 即Activity已经创建完成, activity的资源也已经全部可用.
> * onStart() : 调用这个方法时,就表明Fragment对用户可见了, fragment的onStart是与Activity的onStart绑定的
> * onResume() : 与Activity的onResume类似, 调用这个方法时,表明fragment对用户可见, 当然fragment的onResume方法时基于Activity 的onResume方法的. 当onResume方法结束后,就可以与用户交互了
> * onPause() : 与Activity的onPause绑定,并且作用也是类似
> * onStop() : 与Activity的onStop绑定, 并且类似,表明fragment不在对用户可见
> * onDestroyView() : 当之前在onCreateView中创建的视图与fragment分离时调用, 如果下一次fragment想展示,就会创建一个新的视图, 该方法在onStop之后, onDestroy之前调用.无论onCreateView是否返回非空视图,都会调用该方法. 它会在视图状态保存后, 在它被父视图移除之前被调用
> * onDestroy() : 当这个fragment不再使用时调用.
> * onDetach() : 取消fragment与activity的关联, 调用完后,不再拥有视图层次结构,资源也都将被释放.

### Fragment的使用

#### 静态方式
静态方式添加就是在布局文件中添加fragment
新建一个项目LearnFragment, 添加一个布局first_fragment.xml
``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#009688">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:text="This is first fragment"
        android:gravity="center"/>
</LinearLayout>
```
然后创建绘制该视图的java类
``` java
package com.example.hao.learnfragment.fragments;

import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

import com.example.hao.learnfragment.R;

/**
 * Created by hao on 17-4-20.
 */

public class FirstFragment extends Fragment {

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.first_fragment, container, false);
        return view;
    }
}
```
如法炮制,再创建second_fragment.xml和SecondFragment类, 然后修改MainActivity的布局文件activity_main,xml文件
``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <fragment
        android:id="@+id/id_first_fragment"
        android:name="com.example.hao.learnfragment.fragments.FirstFragment"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        tools:layout="@layout/first_fragment" />

    <fragment
        android:id="@+id/id_second_fragment"
        android:name="com.example.hao.learnfragment.fragments.SecondFragment"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        tools:layout="@layout/second_fragment" />

</LinearLayout>
```
因为我们使用的时v4下的Fragment, 所以MainActivity应该继承v4的FragmentActivity, 但是,API22之后就强制使用AppCompatActivity类,而该类已经继承了FragmentActivity, 故对于MainActivity不需要修改

感受下效果图
![Alt text](/assets/images/fragment/learnFragmentTwoFragmentsShow.png)

#### 动态添加
当然我们也可以通过动态添加的方式在代码中添加fragment, 这样做的好处是能够在Activity运行期间随时将fragment添加到Activity布局中;
如果想在Activity中执行对fragment的操作,比如添加,移除或替换等, 可以分为几个步骤
> * 获取FragmentManager实例, v4包中的获取方法通过getSupportFragmentManager方法
> * 开启事务, 通过FragmentManager实例的beginTransaction方法
> * 向事务中添加容器id和fragment
> * 调用commit方法使更改生效

动态添加代码为
``` java
FragmentManager fragmentManager = getSupportFragmentManager();
FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
ExampleFragment fragment = new ExampleFragment();
fragmentTransaction.add(R.id.fragment_container, fragment);
fragmentTransaction.commit();
```

还是修改上面的例子, activity_main.xml
``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:id="@+id/id_button_first_fragment"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="First Fragment"/>

    <Button
        android:id="@+id/id_button_second_fragment"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Second Fragment"/>

    <FrameLayout
        android:id="@+id/frame_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```
然后再修改下MainActivity中的代码
``` java
package com.example.hao.learnfragment;

import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentTransaction;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;

import com.example.hao.learnfragment.fragments.FirstFragment;
import com.example.hao.learnfragment.fragments.SecondFragment;

public class MainActivity extends AppCompatActivity implements View.OnClickListener{

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button firstFragment = (Button)findViewById(R.id.id_button_first_fragment);
        firstFragment.setOnClickListener(this);

        Button secondFragment = (Button)findViewById(R.id.id_button_second_fragment);
        secondFragment.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        FragmentManager fragmentManager = getSupportFragmentManager();
        FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
        switch (v.getId()){
            case R.id.id_button_first_fragment:
                Fragment firstFragment = new FirstFragment();
                fragmentTransaction.add(R.id.frame_layout, firstFragment);
                break;
            case R.id.id_button_second_fragment:
                SecondFragment secondFragment = new SecondFragment();
                fragmentTransaction.add(R.id.frame_layout, secondFragment);
                break;
            default:
        }
        fragmentTransaction.commit();
    }
}
```
展示图
![Alt text](/assets/images/fragment/Fragment动态添加展示图1.png) ![Alt text](/assets/images/fragment/Fragment动态添加展示图2.png)

### 后记
本部分主要记录了Fragment的设计原理,生命周期,以及基本使用方法, 还有很多关于Fragment的知识并没有说到,希望自己能够多加练习关于Fragment的使用, 也期待能够学习后续的知识,继续整理,前进.