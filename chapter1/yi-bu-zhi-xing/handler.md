[TOC]
# Handler
## 为什么要引入Handler
为了解决在子线程不能更新UI的问题。
## Handler原理分析
因为一个线程要想使用Handler必须得创建一个Looper对象，所以首先从Looper讲起，Looper中有两个很重要的方法，一个是Looper.prepare()，这个方法会调用Looper的构造方法，创建出一个Looper对象，与此同时会在它的构造方法中生成一个消息队列的对象MessageQueue，生成的Looper对象会放入ThreadLocal中，ThreadLocal用于对Looper对象进行管理，保证每个线程有且只有一个Looper对象，另一个方法是Looper.loop()，这个方法用于开启消息循环，在这个方法内部，使用一个无限循环，不断地从消息队列取出消息，然后调用Handler的dispatchMessage()方法，
