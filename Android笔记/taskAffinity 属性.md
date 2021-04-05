### taskAffinity 属性

- 任务：执行某一确定任务时与用户交互的 Activity 集合（有可能不同 APP，可以跨进程）
  - 每个 Activity 运行时都有一个其归属的 task 栈，如果两个 activity 的 taskId 不同，则不属于同一个 task
- 返回栈：按照 Activity 打开顺序，Activity 被排列到一个堆栈
- taskAffinity 属性：
  - 单独对一个 activity 使用，代表该 activity 归属的 task
  - 对 application 使用，代表该 application 内所有 activity 都归属于这个 task
  - 默认值：manifest 声明的包名（package 对应字符串）
  - Android 手机的任务列表是根据不同 task 弹出的，可根据任务管理器有几个 item 图标知道开启了几个 task
  - Activity 运行时所归属 task 默认是启动它的 Activity 所在的 task
  - taskAffinity 单独使用不生效，需配合其他属性（Intent.FLAG_ACTIVITY_NEW_TASK
    、allowTaskReparenting）
    - Intent.FLAG_ACTIVITY_NEW_TASK 解析：AMS 发现启动一个带有 FLAG_ACTIVITY_NEW_TASK 标签的 Activity 时会先寻找当前是否存在 Activity 的 task 值，不存在就创建该 task，然后 Activity 添加到 task 中
    - 新开 task 的条件：FLAG_ACTIVITY_NEW_TASK 和 taskAffinity 不同
  - activity 的 taskAffinity 为空字符串，表明 activity 不属于任何 task

```xml
<application
             android:name="MyApplication"
             android:taskAffinity="xxx.xxx.xxx">
    <activity android:name="MainActivity"
              android:taskAffinity="zzz.zzz.zzz">
</application>
```

#### （1）<activity\> 属性

- android:allowTaskReparenting：
  - 标记 Activity 能否从启动的 Task 移动到有 affinity 的 Task（当这个Task进入到前台时），true 表示能，false 表示必须呆在启动时的 Task 里。 默认值 false
  - 一般当 Activity 启动后就与启动它的 Task 关联，并在这里耗尽整个生命周期。当当前 Task 不再显示，可以使用该特性强制 Activity 移动到有 affinity 的 Task 中
    - PS：如果 email 中包含一个 web 链接，点击会启动一个 Activity 显示页面。Activity 由 Browser 应用程序定义，但现在它作为 email Task 的一部分。如果重新宿主到 Browser Task 里，当 Browser 下一次进入前台时就能被看见，且当 email Task 再次进入前台时，就看不到了
  - Task 的 affinity 通过读取根 Activity 的 affinity 决定。因此根 Activity 总位于相同 affinity 的 Task 里。启动模式为 singleTask 和 singleInstance 的 Activity 只能位于 Task 底部，因此重新宿主只限于 standard 和 singleTop 模式 
- android:alwaysRetainTaskState：
  * 标记 Activity 所在 Task 状态是否总由系统保持。 true 表示总是；false 表示在某种情形下允许系统恢复 Task 到其初始化状态。默认值 false。该特性只针对 Task 的根 Activity
  * 一般特定的情形如当用户从主画面重新选择这个 Task 时，系统会对这个 Task 进行清理（从 stack 中删除位于根 Activity 之上所有 Activivity）。当用户有一段时间没有访问这个 Task 时也会这么做。 当该特性设为 true 时，用户总能回到该 Task 的最新状态，无论如何启动。如 Browser 应用有很多个打开的 Tab，用户不想丢失这些状态

- android:clearTaskOnLaunch：
  - 标记是否从 Task 中清除所有的 Activity，除根 Activity 外（每当从主画面重新启动时），true 表示总是清除至根 Activity，false 表示不。默认 false，该特性只针对启动一个新 Task 的 Activity（根 Activity） 
  - 为 true 每次用户重新启动 Task 时都会进入到根 Activity 中，不管 Task 最后在做什么，不管用户使用 BACK 还是 HOME 离开的。为 false 时可能会在一些情形下清除 Task 的 Activity，但不总是

- android:finishOnTaskLaunch： 
  - 标记当用户再次启动它的 Task（在主画面选择这个 Task）时已经存在的 Activity 实例是否要关闭，true 表示应该关闭，false 表示不关闭，默认 false。如果该特性和 allowTaskReparenting 都设为 true，该特性优先。Activity 的 affinity 忽略，Activity 不会重新宿主，但会销毁