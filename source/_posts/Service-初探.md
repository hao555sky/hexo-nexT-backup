---
title: Service 初探
date: 2016-12-08 16:25:49
tags: Android
categories: Android
---

### Service 初探
[TOC]
#### Service
Service 是一个在后台长时间运行并执行操作而不使用用户界面的应用组件。Service没有用户界面，摆明了就是不让用户操作，但是它怎么工作呢？
原来Service可以由组件启动或者将组件绑定到Service，进而执行相关操作，并且这一切全都在后台进行。

#### Service 工作形式
##### 启动
> * 启动方式: 应用组件调用startService()，Service即处于启动状态。
> * 生命周期: Service 在后台无限期运行，即使启动服务的组件被销毁也不受影响。
> * 存在形式: 启动的服务通常执行单一操作，且不会将结果返回给调用方。
> * 通信方式: startService()传递的Intent
> * 结束方式: 服务使用stopSelf() 或者由其他组件调用stopService()停止。

<!-- more -->

##### 绑定
> * 启动方式: 应用组件调用bindService()，Service即处于绑定状态。
> * 生命周期: 多个组件可同时绑定到该服务，只有当所有的组件取消绑定后，服务才会被销毁。
> * 存在形式:
> * 通信方式: 绑定服务提供了一个客户端-服务器接口，允许组件与服务进行交互、发送请求、获取结果甚至是利用进程                     间通信(IPC)跨进程执行这些操作。
> * 结束方式: 取消该服务与所有客户端之间的绑定，系统便会销毁该服务

注意
1. 服务也可以同时以两种方式运行，即它可以是启动服务(无限期运行)，也允许绑定。关键在于是否实现了回调方法: onStartCommand()（允许组件启动服务）和onBind()（允许绑定服务）。
2. 服务默认在其托管进程的主线程中运行，它既不创建自己的线程，也不在单独的进程中运行（除非另行指定）。也就表明，若服务将执行某些耗时操作，则应创建新线程执行，可以让主线程专注于用户与Activity之间的交互。

#### 如何创建Service
##### 创建Service的子类（或使用现有子类，例如IntentService）。重写一些关键的回调方法，要重写的回调方法包括
* onStartCommand()
> 当另一个组件（如Activity）调用startService()启动服务时，系统调用此方法。若只想提供绑定，则无需实现此方法。

*  onBind()
> 当另一个组件调用bindService()与服务绑定时，系统调用此方法。在此方法的实现中，返回IBinder对象，供客户端与服务进行通信。如果不希望绑定，应返回null。

*  onCreate()
> 在每个service的生命周期中只会被调用一次，并且在onStartCommand()以及onBind()之前，故可在此方法中进行一些一次性的初始化工作。

*  onDestroy()
> 这是服务调用的最后一个方法。当这个方法被调用后，服务将不再使用且被销毁，故可在此方法中清理所有资源，比如线程、注册的侦听器等。

##### 使用清单文件声明服务
同其他组件一样，要在manifest文件中声明所有服务。即添加`<service>`元素作为`<application>`元素的子元素
`<service>`元素包含众多属性，来定义一些特性。android:name 属性是唯一必须的属性，用于指定服务的类名
*android相关属性*

``` java
<service android:description="string resource"
         android:enabled=["true" | "false"]
         android:exported=["true" | "false"]
         android:icon="drawable resource"
         android:isolatedProcess=["true" | "false"]
         android:label="string resource"
         android:name="string"
         android:permission="string"
         android:process="string" >
    . . .
    </service>
```
*注意*
* 为了保证应用的安全性，尽量使用显示Intent启动或者绑定Service，且不要声明Intent filter.
* 尽量添加android:exported=false属性，确保服务仅适用你的应用。

##### 创建启动服务
一般来说，可以扩展两个类来创建启动服务:
> * Service
>  适用于所有服务的基类。扩展此类时，默认情况，服务将使用应用的主线程，会降低正在运行的所有Activity的性能。故必须创建一个用于执行所有服务工作的新线程。

> * IntentService
>  这是 Service 的子类，它使用工作线程逐一处理所有启动请求。如果您不要求服务同时处理多个请求，这是最好的选择。 您只需实现 onHandleIntent() 方法即可，该方法会接收每个启动请求的 Intent，使您能够执行后台工作。

###### 扩展Service类
``` java
public class CommonServiceDemo extends Service {
    private static final String TAG = "CommonServiceDemo";

    @Override
    public void onCreate(){
        super.onCreate();
        Log.d(TAG, "onCreate");
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId){
        Log.d(TAG, "onStartCommand");
        return super.onStartCommand(intent, flags, startId);
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent){
        Log.d(TAG, "onBind");
        return null;
    }

    @Override
    public void onDestroy(){
        Log.d(TAG, "onDestroy");
        super.onDestroy();
    }
}

```
###### Manifest文件
    <service android:name=".ServiceDemo" />

###### 扩展IntentService类
如果启动服务不需要同时处理多个请求（实际上，这种多线程情况可能很危险），所以使用IntentService类实现服务是最好的选择。
IntentService操作:
* 创建默认的工作线程，用于在应用的主线程外执行传递给onStartCommand()的所有Intent
* 创建工作队列，用于将Intent逐一传递给onHandleIntent()实现，这样就不必担心多线程问题
* 处理完所有启动请求后停止服务，因此不必调用stopSelf()
* 提供onBind()的默认实现（返回null）
* 提供onStartCommand()的默认实现，可将Intent一次发送到工作队列和onHandleIntent()实现
2
zhixu实现一个构造函数和onHandleIntent()方法即可
``` java
public class HelloIntentService extends IntentService {

    public HelloIntentService(){
        super("HelloIntentService");
    }

    @Override
    protected void onHandleIntent(Intent intent){
        try {
            Thread.sleep(5000);
        }catch (InterruptedException e){
            Thread.currentThread().interrupt();
        }
    }
}
```

##### 启动服务
在Activity或者其他应用组建中，可通过Intent传递给startService()启动服务。Android 系统调用服务的 onStartCommand() 方法，并向其传递 Intent。（切勿直接调用 onStartCommand()。）

例如，显示Intent传递给startService，启动服务CommanServiceDemo
``` java
Intent intent =  new Intent(MainActivity.this, CommonServiceDemo.class);
startService(intent);
```
*注意*
* 如果服务尚未运行，则系统会先调用 onCreate()，然后再调用 onStartCommand()。
* 多个服务启动请求会导致多次对服务的 onStartCommand() 进行相应的调用。但是，要停止服务，只需一个服务停止请求（使用 stopSelf() 或 stopService()）即可。

##### 创建绑定服务
要创建绑定服务，必须实现onBind()方法，返回IBinder对象，作为客户端与服务通信的接口。

多个客户端可以同时绑定到服务。客户端完成与服务的交互后，会调用 unbindService() 取消绑定。一旦没有客户端绑定到该服务，系统就会销毁它。

绑定服务的实现比启动服务更加复杂，故放到下一篇博客

### 结语
本篇博客主要讲解了service的一些基本知识，包括什么是service，service的类别，如果创建service，如何启动service，关于service中的绑定服务，生命周期，以及启动服务的扩展Service（执行多线程而不是通过工作队列处理启动请求），前台运行的相关知识还未讲解，故希望日后能都学习补全。