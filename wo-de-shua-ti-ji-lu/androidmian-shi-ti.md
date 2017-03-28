## 1.Android的Activity什么时候会调用onCreate\(\)而不调用onStart\(\)?

* 直接在onCreate里面finish\(\);

* 获取当前进程的id，使用Android.os.Process.killProcess\(android.os.Process.myPid\(\)\);杀死进程，可能在后台留有缓存

* 终止当前正在运行的java虚拟机，System.exit\(0\)，可以同时清除后台缓存的本进程，System.exit\(x\);x = 0表示正常退出，x != 0表示异常退出，x参数只是用来通知操作系统该程序是否是正常退出。

* 强制关闭与该包关联的一切执行：

```java
ActivityManager manager = (ActivityManager)getSystemService(Context.ACTIVITY_SERVICE);
manager.restartPackage(getPackageName());
```

使用这个方法需要加入权限&lt;uses-permission android:name = "android.permission.RESTART\_PACKAGES"/&gt;

这几种方法都是有缺陷的，都不能完全退出程序，第二种方式：他不会把当前应用程序的activity的task栈清空。第三种方式：它只能杀死其他应用程序而不能杀死自己的。

## 2.引入内部类要解决的问题是什么？

* 一个类可以访问另一个类的私有成员\(封装性\)
* 让java可以实现多继承

## 3.内部类如何访问外部类的成员？

1. 编译器自动为内部类添加一个成员变量，这个成员变量的类型和外部类的类型相同，这个成员变量就是指向外部类对象的引用；
2. 编译器自动为内部类的构造方法添加一个参数，参数的类型是外部类的类型，在构造方法内部使用这个参数为1中添加的成员变量赋值；
3. 在调用内部类的构造函数初始化内部类对象时，会传入外部类的引用。

## 4.什么是内存泄漏？

内存泄漏，简单说就是应该被释放的内存没有被释放，一直被某个实例引用但无法使用，导致GC无法回收，造成内存泄露；

总结的说，长生命周期的对象一直持有短生命周期对象的引用，导致短生命周期的对象一直被引用，无法被GC回收。

内存泄漏是造成OOM的主要原因之一，当一个应用中产生的内存泄漏比较多时，就难免会导致应用所需要的内存超过这个系统分配的内存限额，这就会导致应用Crash.

## 5.Android中常见的产生内存泄漏的场景

* 单例造成的内存泄漏

因为单例模式有其静态的特点，其生命周期和应用一样长，如果单例对象中包含了一个其他对象的引用，那么即使这个对象不再使用，依然存在一个单例对象引用它，就会造成无法回收。例如：

```java
public class AppManager{
    private static AppManager instance;
    private Context context;
    private AppManager(Context context){
        this.context = context;
    }
    public static AppManager getInstance(Context context){
        if(instance == null){
            instance = new AppManager(context);
        }
        return instance;
    }
}
```

这个单例对象包含了一个Context的引用，如果传入了一个Application Context它的生命周期和应用一样长，所以不会发生内存泄漏

如果传入一个Activity Context，Activity退出销毁后，但仍然会被单例所引用，所以会导致内存泄漏；所以正确的单例为：

```java
public class AppManager{
    private static AppManager instance;
    private Context context;
    private AppManager(Context context){
        this.context = context.getApplicationContext();
    }
    public static AppManager getInstance(Context context){
        if(instance == null){
            instance = new AppManager(context);
        }
        return instance;
    }
}
```

* 非静态内部类创建其静态实例造成内存泄漏

```java
public class MainActivity extends AppCompatActivity {
    private static TestResource mResource = null;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        if(mResource == null){
            mResource = new TestResource();
        }
        //...
    }
    class TestResource {
        //...
    }
}
```

这样就在Activity内部创建了一个非静态内部类的单例，每次启动Activity时都会使用该单例的数据，这样虽然避免了资源的重复创建，不过这种写法却会造成内存泄露，因为非静态内部类默认会持有外部类的引用，而又使用了该非静态内部类创建了一个静态的实例，该实例的生命周期和应用的一样长，这就导致了该静态实例一直会持有该Activity的引用，导致Activity的内存资源不能正常回收。

正确的做法为：将该内部类设为静态内部类或将该内部类抽取出来封装成一个单例，如果需要使用Context，请使用Application Context

* 匿名内部类/异步线程造成内存泄漏

```java
//——————test1
        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... params) {
                SystemClock.sleep(10000);
                return null;
            }
        }.execute();
//——————test2
        new Thread(new Runnable() {
            @Override
            public void run() {
                SystemClock.sleep(10000);
            }
        }).start();
```

上面的异步任务和Runnable都是一个匿名内部类，因此他们对当前的Activity都有一个隐式引用。如果在Activity在销毁之前，任务还未完成，那么将导致Activity的内存资源无法回收，造成内存泄露。正确的做法还是使用静态内部类的方式

```java
static class MyAsyncTask extends AsyncTask<Void, Void, Void> {
        private WeakReference<Context> weakReference;
        public MyAsyncTask(Context context) {
            weakReference = new WeakReference<>(context);
        }
        @Override
        protected Void doInBackground(Void... params) {
            SystemClock.sleep(10000);
            return null;
        }
        @Override
        protected void onPostExecute(Void aVoid) {
            super.onPostExecute(aVoid);
            MainActivity activity = (MainActivity) weakReference.get();
            if (activity != null) {
                //...
            }
        }
    }
    static class MyRunnable implements Runnable{
        @Override
        public void run() {
            SystemClock.sleep(10000);
        }
    }
//——————
    new Thread(new MyRunnable()).start();
    new MyAsyncTask(this).execute();
```

这样就避免了Activity的内存资源泄漏，当然在Activity销毁时，也应该取消相应的任务AsyncTask::cancel\(\);避免任务在后台执行浪费资源。

* Handler机制造成的内存泄漏

  [Handler](/chapter1/yi-bu-zhi-xing/handler.md)

```java
public class MainActivity extends AppCompatActivity {
    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            //...
        }
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        loadData();
    }
    private void loadData(){
        //...request
        Message message = Message.obtain();
        mHandler.sendMessage(message);
    }
}
```

其中mHandler作为一个非静态匿名内部类，持有一个外部类—MainActivity的引用，我们知道对于消息机制是Looper不断的轮询从消息队列取出未处理的消息交给handler处理，而对于这个例子，每一个消息又持有一个mHandler的引用，每一个mHandler又持有MainActivity的引用，所以如果在Activity退出后，消息队列中还存在未处理完的消息，导致该Activity一直被引用，其内存资源无法被回收，导致了内存泄漏。一般我们使用静态内部类和弱引用的写法写Handler。

```java
public class MainActivity extends AppCompatActivity {
    private MyHandler mHandler = new MyHandler(this);
    private TextView mTextView ;
    private static class MyHandler extends Handler {
        private WeakReference<Context> reference;
        public MyHandler(Context context) {
            reference = new WeakReference<>(context);
        }
        @Override
        public void handleMessage(Message msg) {
            MainActivity activity = (MainActivity) reference.get();
            if(activity != null){
                activity.mTextView.setText("");
            }
        }
    }
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mTextView = (TextView)findViewById(R.id.textview);
        loadData();
    }
    private void loadData() {
        //...request
        Message message = Message.obtain();
        mHandler.sendMessage(message);
    }
}
```

Java对引用的分类有强引用，软引用，弱引用，虚引用

| 级别 | 回收时机 | 用途 | 生存时间 |
| :---: | :---: | :---: | :---: |
| 强 | 从来不会 | 对象的一般状态 | JVM停止运行时终止 |
| 软 | 在内存不足时 | 联合ReferenceQueue构造有效期短/占内存大/生命周期长的对象的二级高速缓冲器（内存不足时才清空） | 内存不足时终止 |
| 弱 | 在垃圾回收时 | 联合ReferenceQueue构造有效期短/占内存大/生命周期长的对象的一级高速缓冲器（系统发生GC则清空） | GC回收时终止 |
| 虚 | 在垃圾回收时 | 联合ReferenceQueue来跟踪对象被垃圾回收器回收的活动 | GC回收时终止 |

在Android应用开发中，为了防止内存溢出，在处理一些占用内存大而且生命周期较长的对象时候，可以尽量应用软引用和弱引用技术。

* 资源未回收导致内存泄漏

## **总结**

1. Activity等组件的引用应该控制在Activity的生命周期之内，如果不能就考虑使用getApplicationContext或者getApplication，以避免Activity被外部长生命周期的对象引用而泄漏。
2. 对于生命周期比Activity长的内部类对象，并且内部类中使用了外部类的成员变量，可以这样做避免内存泄漏：将内部类改为静态内部类，静态内部类中使用弱引用来引用外部类的成员变量。
3. Handler的持有的引用对象最好使用弱引用，资源释放时也可以清空Handler里面的消息。例如在Activity的onStop或者onDestory的时候，取消掉该Handler对象的Message和Runnable
4. 在java的实现过程中，也要考虑其对象的释放，最好的方法是在不使用某对象时，显式地将此对象对象赋值为null，比如使用完Bitmap后先调用recycle\(\)，在赋值为null，清空对图片等资源有直接引用或者间接引用的数组（使用array.clear\\(\\);array = null）等，最好遵循谁创建谁释放的原则。
5. 正确关闭资源，对于使用了BroadcastReceiver，ContentObserver，File，游标Cursor，Stream，Bitmap等资源的使用，应该在Activity销毁时及时关闭或者注销。
6. 保持对对象生命周期的敏感，特别注意单例、静态对象、全局性集合等的生命周期。

## 6.关于Broadcast和BroadcastReceiver

### 6.1自定义广播接收器的两种方式

1. 静态注册

```java
<receiver android:name=".BroadcastReceiver1" >
    <intent-filter>
        <action android:name="android.intent.action.CALL" >
        </action>
    </intent-filter>
</receiver>
```

2.动态注册

```java
receiver = new BroadcastReceiver();
IntentFilter intentFilter = new IntentFilter();
intentFilter.addAction(CALL_ACTION);
context.registerReceiver(receiver, intentFilter);
```

区别：如果在代码中注册，app退出后就接收不到广播

自Android3.1开始，系统本身则增加了对所有app当前是否处于运行状态的跟踪，在发送广播时，不管是什么广播类型，系统默认直接增加了值为FLAG\_EXCLUDE\_STOPPED\_PACKAGES的flag，导致即使是静态注册的广播接收器，对于其所在进程已经退出的app，同样导致无法接收到广播。

### 6.2BroadCastReceiver的生命周期

广播接收者的生命周期非常短暂，在接收到广播的时候创建，onReceiver\(\)方法结束之后销毁；

广播接收者中不要做一些耗时的工作，否则会弹出ANR\(Application No Response\)错误对话框；

最好也不要在广播接收者中创建子线程做耗时的工作，因为广播接收者被销毁后进程就成为了空进程，很容易被系统杀掉；

耗时的较长的工作最好放在服务中完成

### 6.3为什么要在Android中引入广播机制（待完善）？

* 从 MVC 的角度考虑\(应用程序内\) 其实回答这个问题的时候还可以这样问, android 为什么要有那 4 大组件,现在的移动开发模型基本上也是照搬的 web 那一套 MVC 架构,只不过是改了点嫁妆而已。android 的四大组件本 质上就是为了实现移动或者说嵌入式设备上的 MVC 架构,它们之间有时候 是一种相互依存的关系,有时候又是一种补充关系,引入广播机制可以方便 几大组件的信息和数据交互。
* 程序间互通消息\(例如在自己的应用程序内监听系统来电\)
* 效率上\(参考 UDP 的广播协议在局域网的方便性\)
* 设计模式上\(反转控制的一种应用,类似监听者模式\)

### 6.4如何让自己的广播只让指定的app接收？

通过自定义广播权限来保护自己发出的广播。 在清单文件里receiver必须有这个权限才能收到广播。 首先，需要定义权限： 然后，声明权限： 这时接收者就能收到发送的广播。

### 6.5广播的优先级对无序广播生效嘛？

生效

### 6.6动态注册的广播优先级谁高

谁先注册谁优先级高

### 6.7如何判断当前BroadCastReceiver接收到的是有序广播还是无序广播？

在 BroadcastReceiver 类中 onReceive\(\)方法中,可以调用 boolean b = isOrderedBroadcast\(\);判断接收到的广播是否为有序广播。

### 6.8粘性广播有什么用？怎么使用？

粘性广播主要为了解决，在发送完广播之后，动态注册的接收者，也能够收到广播。举个例子首先发送一广播，我的接收者是通过程序中的某个按钮动态注册的。如果不是粘性广播，我注册完接收者肯定无法收到广播了。这是通过发送粘性广播就能够在我动态注册接收者后也能收到广播。

> 在 android 5.0/api 21中deprecated,不再推荐使用,包括粘性有序广播

### 6.9增加广播的安全性

* 对于同一App内部发送和接收广播，将exported属性人为设置成false，使得非本App内部发出的此广播不被接收；
* 在广播发送和接收时，都增加上相应的permission，用于权限验证；
* 发送广播时，指定特定广播接收器所在的包名，具体是通过intent.setPackage\(packageName\)指定，这样此广播将只会发送到此包中的App内与之相匹配的有效广播接收器中。

## 7.java的类加载机制\(待完善\)

## 8.ArrayList和LinkedList的区别\(待完善\)

## 9.MVP架构\(待完善\)

## 10.自定义View

## 11.Volley，用的什么队列。缓存在本地的是个什么东西？

## 12.MVC和MVP

## 13.HashMap

## 14.View的绘制

## 15.Handler的工作原理

## 16.loop死循环一直阻塞吗？（native层管道+epoll）

## 17.锁\(从volatile到syn到自旋锁到自适应自旋锁到逃逸分析到锁细化到锁粗化到轻量锁到偏向锁\)

## 18.内存模型

## 19.内存分配

## 20.静态常量池和运行时常量池

## 21.反射机制

## 22.类加载

## 23.单链表成环

## 24.ListView和RecycleView

## 25.多种类型ItemTypeImageLoader

## 26.结合情景问Activity生命周期

## 27.结合情景问启动模式SingleTask

## 28.问Volley返回的Response是啥?

## 29.发起一个网络请求，说一个域名，整个过程（DNS解析，TCP链接，请求）

## 30.DNS解析时域名服务器中没有找到IP地址会报啥错，

## 31.TCP三次握手，为什么是三次？

## 32.可靠还有哪些机制？\(流量控制，超时重传，拥塞控制，慢启动\)

## 33.Handler中怎么让Looper阻塞?

## 34.ConcurrentHashMap

## 35.synchronized\(class\),synchronized\(Object\)的区别

## 36.volatile和synchronized的区别

## 37.TCP和UDP的区别

## 38.http和http，https的过程

## 39.对称加密和非对称加密

## 40.堆排

## 41.大数相乘算法

## 42.session工作机制

## 43.怎么优化快排？

## 44.粘包问题

## 45.断点续传

## 46.文件传输发送端发了1024个字节，接收端接收不到1024个字节，为什么？

## 47.MTU

## 48.WebView内存泄漏

## 49.操作系统有哪些进程间通信：管道，消息队列，信号量，共享内存。

## 50.Binder工作原理

## 51.302重定向问题

## 52.ViewPager+Fragment的预加载\(采用懒加载方式解决\)

## 53.动画

## 54.304,http缓存

## 55.http协议栈下面是啥？

## 56.IP协议是什么?

## 57.NAT是什么?

## 58.bug反馈，如何及时捕捉到用户错误，哪个第三方库？

## 59.C语言为什么叫函数，而java叫方法

## 60.重写一个类的equals\(\)方法，为什么必须也要重写hashCode\(\)方法

## 61.OSI七层模型

## 62.线程和进程的区别

## 63.找最大的第k个数

## 64.怎么解决hash冲突

## 65.平衡二叉树

## 66.cookie和session的区别，cookie什么用，session在服务端存储到哪里？

## 67.http缓存机制

## 68.OSI网络模型分几层？哪几层？作用是什么？常见的协议都在哪些层？为什么要分七层？对一个不懂计算机的用户有什么意义？

## 69.get和post的区别

## 70.Http状态码

## 71.ListView和RecycleView的区别和优化

## 72.事件分发机制

## 73.http的一次请求过程

## 74.hashMap、LinkedHashMap、CurryMap

## 75.viewPager缓存

## 76.红黑树

## 77.二叉树广度优先遍历和深度优先遍历

## 78.哈夫曼编码和哈夫曼树

## 79.死锁产生的原因

## 80.如何自定义View

## 81.AsyncTask

## 82.指针和引用的区别

## 83.如何判断单向链表是否构成环

## 84.Volley的优缺点

## 85.LRU算法

## 86.抽象类和接口的区别

## 87.java内存泄露的原因，实际开发中遇到的内存泄漏的问题

## 88.单例模式的安全性，双重锁的位置

## 89.ImageLoader的缓存实现

## 90.LruCache的实现

## 91.TCP的拥塞

## 92.缺页中断

## 93.ArrayList的实现原理

## 94.图片的双缓存怎么处理?

## 95.程序为什么会发生OOM？怎么解决?

## 96.dp,sp,px的区别

## 97.margin和padding的区别

## 98.使用多种方式实现一个圆角的按钮

## 99.实现下拉刷新的多种方式

## 100.View中使用View.post\(Runnable\)和Handler更新UI有什么区别？

## 101.service的onStartCommand方法执行在哪个线程，intentService中线程怎么实现的？

## 102.ArrayList和LinkedList的区别

## 103.如何判断两个链表是否有交点？

## 104.Activity和Fragment的区别、怎么通信？

## 105.进程间通信，aidl,binder

## 106.service启动方式，运行在哪个线程？

## 107.主线程能不能进行网络请求，I/O操作

## 108.如何解决ANR,OOM？

## 109.线程间通信

## 110.View动画和属性动画

## 111.各种Collection区别、collections.sort\(\)是用什么算法排序的

## 112.几种内部类的区别

## 113.HashMap源码是如何实现插入的

## 114.知道什么排序、时间复杂度、问了快排的时间复杂度为什么是nlog2n

## 115.如何保证线程安全？



