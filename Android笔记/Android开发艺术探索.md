## Android开发艺术探索学习笔记

### 一、Activity的生命周期全面分析

#### 1、典型情况下的生命周期分析

> 从实际使用过程来说，onStart和 onResume、onPause和 onStop这两个配对的回调分别表示不同的意义，onStart和onStop是从Activity是否可见这个角度来回调的，而onResume和 onPause是从Activity是否位于前台这个角度来回调的，除了这种区别，在实际使用中没有其他明显区别。

> Activity 的启动过程的源码相当复杂，涉及 Instrumentation、ActivityThread和ActivityManagerService(下面简称AMS)。简单理解,启动Activity的请求会由Instrumentation来处理，然后它通过Binder向AMS发请求，AMS内部维护着一个ActivityStack 并负责栈内的Activity 的状态同步，AMS通过ActivityThread去同步Activity 的状态从而完成生命周期方法的调用。
>
> 在新Activity启动之前，栈顶的Activity需要先onPause后，新Activity才能启动。