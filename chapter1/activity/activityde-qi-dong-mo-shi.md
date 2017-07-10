#Activity的LaunchMode(启动模式)
在默认情况下，当我们多次启动同一个Activity时，系统会创建多个实例并将它们一一放入任务栈中，当单击back键时，这些Activity的实例回依次销毁，栈为空时，系统就会回收这个任务栈，任务栈是一种"后进先出"的栈结构
###四种启动模式
1. standard：标准模式，每次启动一个Activity都会重新创建一个新的实例，不管这个实例之前是否已经存在，一个任务栈可以有多个相同的实例，每个实例也可以属于不同的任务栈。在这种模式下，谁启动了这个Activity，那么这个Activity就运行在启动它的那个Activity所在的任务栈中；<br>
注意：当使用ApplicationContext去启动standard模式的Activity的时候会报错，
```java
Calling startActivity from outside of an Activity Context requires the FLAG_ACTIVITY_NEW_TASK flag.Is this really what you want?
```
这是因为standard模式的Activity默认会进入启动它的Activity所属的任务栈中，但是由于非Activity类型的Context并没有任务栈，所以就产生了问题，解决办法是给这个待启动的Activity添加一个FLAG_ACTIVITY_NEW_TASK标记位，这样这个活动在启动时就会创建一个新的任务栈(类似于singleTask模式进行启动)
2. singleTop：栈顶复用模式，
