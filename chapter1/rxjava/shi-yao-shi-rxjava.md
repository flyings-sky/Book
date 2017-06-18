RxJava是Reactive Extension\(具有可观察流的异步编程API\)的JVM实现:是一个通过使用可观察序列来组合异步和基于事件程序的第三方库。  
为了支持数据/事件序列扩展了观察者模式，添加了允许你以声明方式去组合序列的操作符，与此同时把对低级线程，同步，线程安全和并发数据结构等事物的关注抽象化。

# 入门

## 1.将RxJava2导入到你的项目中:

1.1  如果是Java项目,则需要自己到下面的Maven仓库中下载Jar包自己导入到项目中:

[reactive-streams](https://mvnrepository.com/artifact/org.reactivestreams/reactive-streams/1.0.0)

[RxJava](https://mvnrepository.com/artifact/io.reactivex.rxjava2/rxjava/2.1.0)

1.2  如果是Android项目，添加下列依赖即可使用:

```java
compile "io.reactivex.rxjava2:2.x.y"(x、y用版本号替代)
compile 'io.reactivex.rxjava2:rxandroid:2.x.y'(x、y用版本号替代)
```

## 2.使用RxJava2写一个HelloWorld程序

```java
//未使用Lambda表达式简化的代码
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
```

```java
//最终经过Lambda表达式简化的代码
public class Test {
    public static void main(String[] args) {
        Flowable.
                just("Hello world").
                subscribe(System.out::println);
    }
}
```

RxJava的常见用法是在后台线程中进行网络请求或者进行一些计算，然后在UI线程中显示后台操作的结果。

```java
//未使用Lambda表达式
public class TestCommonUsage {
    public static void main(String[] args) {
        Flowable
                .fromCallable(new Callable<String>() {
            @Override
            public String call() throws Exception {
                //执行耗时操作
                Thread.sleep(1000);
                return "Done";
            }
        })
                .subscribeOn(Schedulers.io())
                .observeOn(Schedulers.single())
                .subscribe(
                        new Consumer<String>() {
                    //执行完耗时操作要进行的操作
                    @Override
                    public void accept(String s) throws Exception {
                        System.out.println(s);
                    }
                }, //耗时操作过程中抛出异常要进行的操作
                        new Consumer<Throwable>() {
                    @Override
                    public void accept(Throwable throwable) throws Exception {
                        throwable.printStackTrace();
                    }
                });
    }
}
```

```java
//使用Lambda表达式
public class TestCommonUsage {
    public static void main(String[] args) {
        Flowable
                .fromCallable(() -> {
                    //执行耗时操作
                    Thread.sleep(1000);
                    return "Done";
                })
                .subscribeOn(Schedulers.io())
                .observeOn(Schedulers.single())
                .subscribe(System.out::println,
                        Throwable::printStackTrace);
    }
}
```



