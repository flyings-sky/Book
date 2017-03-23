[TOC]
# Handler
## 为什么要引入Handler
为了解决在子线程不能更新UI的问题。
## Handler原理分析
因为一个线程要想使用Handler必须得创建一个Looper对象，所以首先从Looper讲起，Looper中有两个很重要的方法，一个是Looper.prepare()，这个方法会调用Looper的构造方法，创建出一个Looper对象，与此同时会在它的构造方法中生成一个消息队列的对象MessageQueue，生成的Looper对象会放入ThreadLocal中，ThreadLocal用于对Looper对象进行管理
