1. Activity、Service、BroadcastReceiver的作用
   Activity：Activity是Android程序与用户交互的窗口，是Android构造块中最基本的一种，它需要为保持各界面的状态，做很多持久化的事情，妥善管理生命周期以及一些跳转逻辑。
   Service：后台服务与Activity，封装有一个完整的功能逻辑实现，接受上层指令，完成相关的指令，定义好需要接受的Intent提供同步和异步的接口。
   BroadCastReceiver：接受一种或者多种Intent作触发事件，接受相关消息，做一些简单处理，转换成一条Notification，统一了Android的事件广播模型。
2. 描述一个完整的Android Activity lifecycle
   activity的生命周期方法有：onCreate\(\)、onStart\(\)、onRestart\(\)、onResume\(\)、onPause\(\)、onStop\(\)、onDestory\(\)
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


8.android中的动画有几种，他们的特点和区别是什么？

三种

* FrameAnimation\(逐帧动画\)：将多张图片组合起来播放，类似于早期电影的工作原理，很多APP的loading是采用这种方式；
* TweenAnimation\(补间动画\)：是对某一个View进行一系列的动画操作，包括淡入淡出\(Alpha\),缩放\(Scale\),平移\(Translate\),旋转\(Rotate\)四种模式
* PropertyAnimation\(属性动画\)：属性动画不再仅仅是一种视觉效果了，而是一种不断地对值进行操作的机制，并将值赋到指定对象的指定属性上，可以是任意对象的任意属性。



9.一条最长的短信息约占多少byte？  
140byte,70个汉字

10.描述Handler机制的原理

android提供了Handler和Looper来满足线程间的通信。

Handler先进先出原则。

Looper类用来管理特定线程内对象之间的消息交换。

1\)Looper：一个线程可以产生一个Looper对象，由它来管理线程里面的消息队列

2\)Handler：可以构造Handler对象来与Looper通信，以便push新消息到消息队列，或者接收Looper从消息队列取出所送来的消息。

3\)Message Queue\(消息队列\)：用来存放线程放入的消息。

4\)线程：UI线程通常是Main Thread，而Android程序启动时会自己在ActivityThread方法里创建一个Looper，这个Looper里面会包含一个消息队列。

11.如何将SQLite数据库\(dictionary.db文件\)与apk一起发布？

可以将dictionary.db文件复制到Eclipse Android工程中的res/raw目录中，所有在res/raw目录中的文件不会被压缩，这样可以直接提取该目录中的文件。

使用openDatabase方法来打开数据库文件，如果文件不存在，系统会自动创建/sdcard/dictionary目录，并将res/raw目录中的dictonary.db文件复制到/sdcard/dictionary目录中。

12.说说android中mvc的具体体现

mvc是model，view，controller的缩写，mvc包含三个部分：

model\(模型\)对象：是应用程序的主体部分，所有业务逻辑都应该写在该层。

view\(视图\)对象：是程序中负责生成用户界面的部分，也是在整个mvc架构中用户唯一可以看到的一层，接收用户的输入，显示处理结果

controller\(控制器\)对象：是根据用户的输入，控制用户界面数据显示及更新model对象状态的部分，控制器更重要的一种导航功能，响应用户发出的相关事件，交给model层处理.

android鼓励弱耦合和组件的重用，在android中mvc的具体体现如下：

1\)视图\(view\)：一般采用xml文件进行界面描述，使用的时候可以非常方便的引入。

2\)控制层\(controller\)：android的控制层的责任通常落在了众多的activity的肩上，这句话也就暗含了不要在activity中写过多的代码，要通过activity交割model业务处理层处理，这样做的另外一个原因是android中的activity的响应时间是5s，如果耗时的操作放在这里，程序很容易被回收掉

3\)模型层\(model\)：对数据库的操作，对网络的操作都应该在model里面处理，当然对业务计算等操作也是必须放在该层。

13.请介绍下Android常用的五中布局
帧布局\(FrameLayout\)
相对布局\(RelativeLayout\)
线性布局\(LinearLayout\)
表格布局\(TableLayout\)
绝对布局\(AbsoluteLayout\)

14.如何启用Service，如何停用Service

1\)startService用于启动Service，stopService停止Service

2\)bindService绑定Service，unbindService解除Service的绑定

15.如何优化ListView？

1\)自定义适配器，在getView方法中要考虑传进来的convertView是否为null，如果为null就创建convertView并返回，如果不为null直接使用，这样可以尽可能少的创建View

2\)给convertView设置tag\(setTag\(\)\)，传入一个ViewHolder对象，用于缓存要显示的数据，可以达到图像数据异步加载的效果。

3\)如果ListView需要显示的item很多，就要考虑分页加载。比如一共要显示100条或者更多数据的时候，我们可以考虑先加载20条，等用户拉到列表底部的时候再去加载接下来的20条。

16.描述4中activity的启动模式

1\)standard：系统默认的模式，每次启动Activity都会生成一个新的实例

2\)singleTop：如果将要启动的Activity在任务栈的栈顶，则不会再生成新的实例，而是调用要启动Activity的onNewIntent的方法，如果不位于栈顶，则会重新创建一个新的实例。

3\)singleTask：先去查找是否存在要启动Activity所需的任务栈，如果找到，则继续查找是否任务栈内存在实例，如果存在，则调用onNewIntent\(\)方法，将该实例上面的所有Activity实例出栈，不产生新的实例，如果找不到所需的任务栈，则新创建一个任务栈，新建一个Activity实例，压入该任务栈中，这种启动模式适用于多个不同的界面会到达同一个界面或者相似界面的情况，例如:qq的显示个人资料的界面，在好友列表和点击聊天的头像都可以进入该界面，所以该界面就可以设计成singleTask模式

4\)singleInstance：设置为singleInstance的activity将独占一个任务栈，独占任务栈与其说是activity，倒不如说是一个应用，这个应用与其他activity是独立的，它有自己的上下文activity

17.什么是Intent，如何使用？

Android基本的设计理念是鼓励减少组件间的耦合，因此Android提供了Intent，Intent提供了一种通用的消息系统，它允许在你的应用程序与其他的应用程序间传递Intent来执行动作和产生事件。使用Intent可以激活Android应用的三个核心组件:活动、服务和广播接收器。

通过startActivity\(\)  or  startActivityForResult\(\)启动一个Activity

通过startService\(\)启动一个服务，或者通过bindService\(\)与后台服务交互

a通过广播方法\(比如sendBroadcast\(\)，sendOrderedBroadcast\(\),sendStickyBroadcast\(\)\)发送给Broadcast Receiver

18.Android用的数据库是什么样的？它和sql有什么区别？为什么要用ContentProvider?它和sql的实现上有什么差别？

Android用的是SQLite数据库。它和其他网络数据库相似，也是通过SQL对数据进行管理，SQLite的操作非常简单，包括数据类型在建表时也可以不指定。

使用ContentProvider可以将数据共享给其他应用，让除本应用之外的应用也可以访问本应用的数据，它的底层使用SQLite数据库实现的，所以其对数据做的各种操作都是以sql实现，只是在上层提供Uri

19.通过Intent传递一些二进制数据的方法有哪些？

1\)使用Serializable接口实现序列化，这是Java的常用方法

2\)实现Parcelable接口，这里Android的部分类比如Bitmap类就实现了，同时Parcelable在Android AIDL中交换数据也很常见。

20.对一些资源以及状态的操作保存，最好是保存在生命周期的那个函数中进行？

onResume\(\)恢复数据,onPause\(\)保存数据

21.如何一次性地退出所有打开的Activity？

编写一个Activity作为入口，当需要关闭程序时，可以利用Activity的SingleTask模式跳转到该Activity，它上面的所有Activity都会被毁掉，然后再将该Activity关闭。或者在跳转时，设置intent.setFlags\(FLAG\_ACTIVITY\_CLEAR\_TOP\);这样就能将上面的Activity销毁掉。

22.说说Service的生命周期？

启动Service的方式有两种，各自的生命周期也有所不同
1)通过startService启动Service：onCreate、onStartCommand、onDestory
2)通过bindService启动Service：onCreate、onBind、onUnbind、onDestory

23.什么是AIDL？AIDL是如何工作的？
AIDL(Android接口描述语言)是一种接口描述语言；编译器可以通过aidl文件生成一段代码，通过预先定义的接口达到两个进程内部通信的目的，如果需要在一个Activity中，访问另一个Service中的某个对象，需要先将对象转换成AIDL可识别的参数(可能是多个参数)，然后使用AIDL来传递这些参数，在消息的接收端，使用这些参数组装成自己需要的对象。AIDL是基于接口的，但它是轻量级的，它使用代理类在客户端和实现层间传递值。
24.Android如何把文件存放在SDCard上?
1)先在AndroidManifest.xml中加入访问SDCard的权限
2)使用Environment.getExternalStorageState()方法用于获取SDCard的状态，如果手机装有SDCard，并且可以进行读写，那么方法返回的状态等于Environment.MEDIA_MOUNTED
3)Environment.getExternalStorageDirectory()方法用于获取SDCard的目录
25.注册广播有几种方式，这些方式有何优缺点？
两种，一种是通过代码注册，这种方式注册的广播会跟随程序的生命周期。第二种是在AndroidManifest.xml中配置广播，这种常驻型广播当应用程序关闭后，如果有信息广播来，程序也会被系统调用自动运行。
26.什么是ANR如何避免它？
在Android上，如果你的应用程序有一段时间响应不够灵敏，系统会向用户显示一个对话框，这个对话框称作应用程序无响应(ANR)对话框,用户可以选择让程序继续运行，但是，他们在使用你的应用程序时，并不希望每次都处理这个对话框。因此，在程序里对响应性能的设计很重要，这样，系统不会显示ANR给用户。要避免它，应尽量少在主线程里做耗时太长操作，应该将这些操作放到线程中去执行。
27.Android本身的api并未声明会抛出异常，则其在运行时有无可能抛出Runtime异常，你遇到过吗？有的话会导致什么问题？如何解决？
有可能比如空指针异常、数组下标越界等异常，这些异常抛出后可能会导致程序Crash，在编写代码时应该做好检测，多考虑可能会发生错误的情况，从代码层次解决这些问题。
28.为什么要用ContentProvider?它和sql的实现上有什么差别？
使用ContentProvider可以将数据共享给其他应用，让除本应用之外的应用也可以访问本应用的数据。它的底层是用SQLite数据库实现的，所以其对数据做的各种操作都是以sql实现，只是在上层提供的是Uri
29.谈谈UI中，Padding和Margin有什么区别？
padding指内边距，表示组件内部元素距离边框的距离。
margin指外边距，表示组件与组件之间的距离。
30.请介绍下Android的数据存储方式。
1)使用SharedPreferences存储数据
2)文件存储数据
3)SQLite数据库存储数据
4)使用ContentProvider存储数据
5)网络存储数据


