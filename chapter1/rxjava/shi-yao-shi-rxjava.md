RxJava是Reactive Extension(具有可观察流的异步编程API)的JVM实现:是一个通过使用可观察序列来组合异步和基于事件程序的第三方库。
为了支持数据/事件序列扩展了观察者模式，添加了允许你以声明方式去组合序列的操作符，与此同时把对低级线程，同步，线程安全和并发数据结构等事物的关注抽象化。
<br>
#入门
1. 将RxJava2导入到你的项目中:
1.1 如果是Java项目,则需要自己到下面的Maven仓库中下载Jar包自己导入到项目中:
<br>
[reactive-streams](https://mvnrepository.com/artifact/org.reactivestreams/reactive-streams/1.0.0)
<br>
[RxJava](https://mvnrepository.com/artifact/io.reactivex.rxjava2/rxjava/2.1.0)

1.2 如果是Android项目，添加下列依赖即可使用:


```java```
compile "io.reactivex.rxjava2:2.x.y"(x、y用版本号替代)
<br>
compile 'io.reactivex.rxjava2:rxandroid:2.x.y'(x、y用版本号替代)

```java```


2.使用RxJava2写一个HelloWorld程序


```java```
public class Test {
    public static void main(String[] args) {
        Flowable.
                just("Hello world").
                subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                System.out.println(s);
            }
        });
    }
}


```java```




