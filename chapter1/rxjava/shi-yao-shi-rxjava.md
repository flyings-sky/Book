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

这种链式调用的形式被称为流式API，书写形式类似于建造者模式。然而，RxJava的反应类型是不可变的\(类似于String，每个方法都会返回一个新的String对象\)，每个方法的调用\(操作符的使用\)都会返回一个添加了某些行为新的Flowable，示例代码如下：

```java
        //另一种形式
        Flowable<String> mFirst = Flowable.fromCallable(() -> {
            Thread.sleep(1000);
            return "done";
        });
        //指定上一步发生的线程
        Flowable<String> mSecond = mFirst.subscribeOn(Schedulers.io());
        //指定下一步发生的线程
        Flowable<String> mThird = mSecond.observeOn(Schedulers.single());
        mThird.subscribe(System.out::println,Throwable::printStackTrace);
```

通常，你可以使用subscribeOn操作符将I/O或者计算操作指定到其他非UI线程执行，一旦这些操作执行完毕，数据准备就绪，你就可以通过observeOn操作符，将线程切换回主线程，进行UI更新操作。

