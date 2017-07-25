1. Activity、Service、BroadcastReceiver的作用
Activity：Activity是Android程序与用户交互的窗口，是Android构造块中最基本的一种，它需要为保持各界面的状态，做很多持久化的事情，妥善管理生命周期以及一些跳转逻辑。
Service：后台服务与Activity，封装有一个完整的功能逻辑实现，接受上层指令，完成相关的指令，定义好需要接受的Intent提供同步和异步的接口。
BroadCastReceiver：接受一种或者多种Intent作触发事件，接受相关消息，做一些简单处理，转换成一条Notification，统一了Android的事件广播模型。
2. 描述一个完整的Android Activity lifecycle
activity的生命周期方法有：onCreate()、onStart()、onRestart()、onResume()、onPause()、onStop()、onDestory()
3. 显式intent和隐式intent的区别是什么？
Intent的定义：intent是一种在不同组件之间传递的请求消息，是应用程序发出的请求和意图。作为一个完整的消息传递机制，Intent不仅需要发送端，还需要接收端
显式Intent定义:对于明确指定了目标组件名称的Intent，我们成为显式Intent
隐式Intent定义：对于没有明确指出目标组件名称的Intent，则称之为隐式Intent
Android系统使用IntentFilter来寻找与隐式Intent相关的对象。
4. Android中线程同步的方法
同步方法和同步块
5. 怎么将一个Activity封装成对话框的样子？怎样将Activity封装成长按Menu菜单的样子？
在AndroidManifest.xml文件里设置Activity的android:theme属性为：

```xml
android:theme = "@android:style/Theme.Dialog"
```
这样就使得Activity以对话框的形式弹出来了

```xml

android:theme = "@android:style/Theme.Translucent"
```
将Activity设置成半透明
重写OnCreateOptionMenu方法处理按下Menu后的行为，然后在该方法中弹出对话框形式的Activity。也可以利用事件监听来监听menu按键，并在该按钮按下后弹出对话框形式的Activity
6. 介绍一下Android系统的体系结构
应用层：android的应用程序通常涉及用户界面和交互
应用框架层：UI组件、各种管理器等
函数库层：系统C层、媒体库、webkit、SQLite等
linux核心库：linux系统运行的组件
7. 描述一下横竖屏切换时activity的生命周期
    * 不设置Activity的android:configChanges属性时，切屏会重新调用各个生命周期，横竖屏切换都执行一次生命周期
    * 设置Activity的android:configChanges属性时，API13以上需要同时配置oriention和screenSize，API13以下，只要配置oriention，切屏时不会重新调用各个生命周期，只会执行onConfigurationChanged方法
8. android中的动画有几种，他们的特点和区别是什么？
三种
    * FrameAnimation(逐帧动画)：将多张图片组合起来播放，类似于早期电影的工作原理，很多APP的loading是采用这种方式；
    * TweenAnimation(补间动画)：是对某一个View进行一系列的动画操作，包括淡入淡出(Alpha),缩放(Scale),平移(Translate),旋转(Rotate)四种模式
    * PropertyAnimation(属性动画)：属性动画不再仅仅是一种视觉效果了，而是一种不断地对值进行操作的机制，并将值赋到指定对象的指定属性上，可以是任意对象的任意属性。
9. 一条最长的短信息约占多少byte？
140byte,70个汉字
10. 描述Handler机制的原理
android提供了Handler和Looper来满足线程间的通信。
Handler先进先出原则。
Looper类用来管理特定线程内对象之间的消息交换。
1)Looper：一个线程可以产生一个Looper对象，由它来管理线程里面的消息队列
2)Handler：可以构造Handler对象来与Looper通信，以便push新消息到消息队列，或者接收Looper从消息队列取出所送来的消息。
3)Message Queue(消息队列)：用来存放线程放入的消息。
4)线程：UI线程通常是Main Thread，而Android程序启动时会自己在ActivityThread方法里创建一个Looper，这个Looper里面会包含一个消息队列。
