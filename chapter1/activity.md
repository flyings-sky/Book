#正常情况下的生命周期
1. onCreate：这是Activity生命周期的第一个方法，表示Activity正在被创建，可以在这个方法里做一些初始化工作；
2. onStart：表示Activity正在被启动，此时Activity已经可见了，但是还没有出现在前台(是否在前台决定了对用户是否可见)，还无法和用户交互；
3. onResume：表示Activity已经可见了，并且出现在前台(可以和用户交互了)；
4. onPause：表示Activity正在停止，当在当前Activity的基础上启动了一个新的非全尺寸或者透明的Activity时(例如：Theme为`Theme.AppCompat.Dialog.Alert`的活动)，该生命周期的回调方法会被执行，暂停状态的活动是完全活着的(它维持所有状态和成员信息，并保持附加到窗口管理器)，但是在极低内存下可能会被系统杀死，因此可以在这里做一些存储数据、停止动画等工作，但是不能太耗时，因为会影响到新Activity界面的显示(先执行完旧Activity的onPause方法，才会执行新Activity的onResume方法)；