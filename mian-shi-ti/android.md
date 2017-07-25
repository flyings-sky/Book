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
函数库 
