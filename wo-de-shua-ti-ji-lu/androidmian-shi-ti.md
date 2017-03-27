1.Android的Activity什么时候会调用onCreate\(\)而不调用onStart\(\)?

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

2.引入内部类要解决的问题是什么？

* 一个类可以访问另一个类的私有成员\(封装性\)
* 让java可以实现多继承

3.内部类如何访问外部类的成员？

1. 编译器自动为内部类添加一个成员变量，这个成员变量的类型和外部类的类型相同，这个成员变量就是指向外部类对象的引用；
2. 编译器自动为内部类的构造方法添加一个参数，参数的类型是外部类的类型，在构造方法内部使用这个参数为1中添加的成员变量赋值；
3. 在调用内部类的构造函数初始化内部类对象时，会传入外部类的引用。

4.什么是内存泄漏？

内存泄漏，简单说就是应该被释放的内存没有被释放，一直被某个实例引用但无法使用，导致GC无法回收，造成内存泄露；

总结的说，长生命周期的对象一直持有短生命周期对象的引用，导致短生命周期的对象一直被引用，无法被GC回收。

内存泄漏是造成OOM的主要原因之一，当一个应用中产生的内存泄漏比较多时，就难免会导致应用所需要的内存超过这个系统分配的内存限额，这就会导致应用Crash.

5.Android中常见的产生内存泄漏的场景

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



