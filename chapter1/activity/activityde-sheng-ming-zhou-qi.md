#正常情况下的生命周期
1. onCreate：这是Activity生命周期的第一个方法，表示Activity正在被创建，可以在这个方法里做一些初始化工作；
2. onStart：表示Activity正在被启动，此时Activity已经可见了，但是还没有出现在前台，还无法和用户交互，在该方法里直接执行finish()方法，会在执行完onStart后直接执行onStop和onDestroy；
3. onResume：表示Activity已经可见了，并且出现在前台(可以和用户交互了)；
4. onPause：表示Activity正在停止(执行完该方法后，当前Activity将不再处于前台(不能直接与用户交互)但仍然可见)，当在当前Activity的基础上启动了一个新的非全尺寸或者透明的Activity时(例如：Theme为`Theme.AppCompat.Dialog.Alert`的活动)，该生命周期的回调方法会被执行，暂停状态的活动是完全活着的(它维持所有状态和成员信息，并保持附加到窗口管理器)，但是在极低内存下可能会被系统杀死，因此可以在这里做一些存储数据、停止动画，将未保存的更改提交给持久数据等工作，但是不能太耗时，因为会影响到新Activity界面的显示(先执行完旧Activity的onPause方法，才会执行新Activity的onResume方法)；
5. onStop：表示Activity即将停止(执行完该方法后Activity处于不可见状态)，此时仍然保留着所有状态和成员信息，因为在执行此方法时，被启动的Activity的onResume方法已经执行完毕(界面已经显示出来)，所以可以做一些稍微重量级的回收工作，但是最好不要太耗时，当其他地方需要内存时，处在该状态的Activity通常会被系统杀死；
6. onDestory：表示Activity即将被销毁，在这里可以做一些回收和资源释放工作；
7. onRestart：表示Activity正在重新启动，一般情况下从不可见状态变为可见状态时，onRestart就会被调用。
![](https://developer.android.com/images/activity_lifecycle.png)
当使用startActivityForResult()方法启动的Activity在运行的过程中突然Crash掉了，那么会在父Activity中接收到一个值为RESULT_CANCELED的resultCode