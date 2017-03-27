# Handler

Handler是Android操作系统中的线程通信工具。

## 为什么要引入Handler

因为在Android中更新UI一般在主线程中执行，而在实际情况中经常要在子线程访问UI。  
**为什么Android系统不允许子线程访问UI？**  
因为Android的UI线程不是线程安全的，如果在多线程操控UI可能会导致UI控件处于不可预知的状态；  
**为什么Android系统不给UI控件加锁呢？**  
首先上锁机制会让UI访问的逻辑变得复杂，其次加锁机制会降低访问效率，因此，Android之所以提供Handler是为了解决在子线程不能更新UI的问题。

## Handler原理分析

因为一个线程要想使用Handler必须得创建一个Looper对象，所以首先从Looper讲起，Looper中有两个很重要的方法，一个是Looper.prepare\(\)，这个方法会调用Looper的构造方法，创建出一个Looper对象，与此同时会在它的构造方法中生成一个消息队列的对象MessageQueue，生成的Looper对象会放入ThreadLocal中，ThreadLocal用于对Looper对象进行管理，保证每个线程有且只有一个Looper对象，另一个方法是Looper.loop\(\)，这个方法用于开启消息循环，在这个方法内部，使用一个无限循环，不断地从消息队列取出消息，然后调用Handler的dispatchMessage\(\)方法，进行消息的处理，这个方法待会儿会详细讲，然后会调用msg.recycleUnchecked\(\)方法将消息进行回收，在创建出Handler对象以后，可以调用sendMessage\(\)/post\(\)方法，发送消息，这两个方法最终都会调用sendMessageAtTime\(\)方法，使消息入队，消息入队以后就可以使用dispatchMessage\(\)方法进行消息的处理，如果是使用post方法进行消息发送的，此时的Message的callback不为空\(Message的callback相当于一个Runnable\)，会调用HandleCallback\(msg\)来处理消息，如果是使用带callback的构造方法生成的Handler对象，那么mCallBack不会为空，则消息会调用mCallBack的handleMessge\(\)方法进行处理，如果上述两个callback都为空，则调用自己实现的handleMessage\(\)方法来处理消息。在子线程中创建Handler对象时，需要显式的写出Looper.prepare\(\)和Looper,loop\(\)方法，而在主线程中，创建Handler对象时，并没有显式的写出这两个方法，是因为在主线程的入口ActivityThread中，会调用Looper.prepareMainLooper\(\)方法，这个方法最终还是会调用looper的prepare方法进行Looper的初始化,然后调用Looper的loop方法，来开启消息循环。

## Looper

在Android中，对于每一个线程都可以创建最多一个Looper对象和多个Handler，Looper对象就像是消息队列的管理者，源源不断的从消息池中拿到消息，交给Handler处理。  
Looper.prepare\(\)确保每个线程只能有一个Looper，其中的参数quitAllowed用于决定当前消息队列是否允许被外部代码终止。

## Message

通常我们都会用obtain\(\)方法去创建Message，如果消息池中有Message，则取出，没有再重新创建。这样可以防止对象的重复创建，节省资源。

## MessageQueue

指的是消息队列，即存放供Handler处理的消息。**MessageQueue主要包括两个操作：插入和读取。读取本身会伴随删除操作。**虽然名字叫队列，但其实内部实现是通过单链表的形式实现的\(单链表在插入和删除上比较有优势，而且不需要一大块连续的存储空间\)

```java
private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mRun = true;
        mThread = Thread.currentThread();
}
```

因为一个线程对应了一个Looper对象，所以可以认为一个线程也对应一个MessageQueue对象,每一个MessageQueue都不能脱离Looper存在.

## ThreadLocal

ThreadLocal是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，数据存储以后，只有在指定线程中可以获取到存储的数据，对于其他线程就获取不到数据。一般来说，当某些数据是以线程为作用域而且不同线程需要有不同的数据副本的时候，可以考虑使用ThreadLocal.比如：对于Handler，它要获取当前线程的Looper，很显然Looper的作用域就是线程，并且不同的线程具有不同的Looper.

```java
public static final void prepare() {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException(“Only one Looper may be created per thread”);
        }
        sThreadLocal.set(new Looper(true));
}
```

## Looper,Handler,MessageQueue的引用关系

一个Handler持有一个消息队列的引用和它构造时所属线程的Looper的引用，也就是说，一个Handler必定有它对应的消息队列和Looper，一个线程至多能有一个Looper和消息队列，但是一个线程可以有多个Handler.  
在主线程中New了Handler对象以后，这个Handler对象自动和主线程自动生成的Looper以及消息队列关联上了。  
在子线程中拿到主线程中Handler的引用，发送消息后，消息对象就会发送到target属性对应的那个Handler对应的消息队列中去，由对应的Looper来取出处理

## Handler的post方法原理

```java
mHandler.post(new Runnable()
        {
            @Override
            public void run()
            {
                Log.e(“TAG”, Thread.currentThread().getName());
                mTxt.setText(“yoxi”);
            }
        });
```

run方法中可以写更新UI的代码，其实这个Runnable并没有创建什么线程，而是发送了一条消息，下面看源码：

```java
public final boolean post(Runnable r)
   {
      return  sendMessageDelayed(getPostMessage(r), 0);
   }
```

最终和Handler.sendMessage一样，调用了sendMessageAtTime，然后调用了enqueueMessage方法，给msg.target赋值为handler，最终加入MessageQueue,此时Message的callback不为空，会执行handleCallback\(\)方法，也就是Runnable里面的run方法。

## Handler中的内存泄漏问题

一般我们写Handler的方式为

```java
Handler mHandler = new Handler(){
    @Override
    public void handleMessage(Message msg){

    }
}
```

当使用内部类\(包括匿名类\)来创建Handler的时候，Handler对象会隐式地持有一个外部类对象\(通常是一个Activity\)的引用，而常常在Activity退出以后，消息队列还有未被处理完的消息，此时activity依然被handler引用，导致内存无法回收而内存泄漏。

## 解决办法

在Handler中增加一个对Activity的弱引用\(WeakReference\)：

```java
static class MyHandler extends Handler{
    WeakReference mActivity;
    MyHandler(Activity acticity){
        mActivity = new WeakReference(activity);
    }
    @Override
    public void handleMessage(Message msg){
        final Activity activity = mActivity.get();
        if(activity != null){
            //
        }
    }
}
```

对于上面的代码，用户在关闭Activity之后，就算后台线程还没结束，但由于仅有一条来自Handler的弱引用指向Activity，所以GC仍然会在检查的时候把Activity回收。这样，内存泄漏的问题就不会出现了。

Handler运行在主线程中\(UI线程中\)，它与子线程可以通过Message对象来传递数据，这个时候，Handler就承担着接受子线程传过来的（子线程用sendMessage\(\)和post\(\)方法传递的）Message对象（里面包含数据）的任务，把这些消息放入主线程队列中，配合主线程进行更新UI。

## 总结

* 通过Looper的prepare方法创建MessageQueue
* 通过loop方法找到和当前线程匹配的Looper

```java
public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;
        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }
            final long traceTag = me.mTraceTag;
            if (traceTag != 0) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }
            try {
                msg.target.dispatchMessage(msg);
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            }
            msg.recycleUnchecked();
        }
    }
```



