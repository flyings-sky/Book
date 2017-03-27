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



