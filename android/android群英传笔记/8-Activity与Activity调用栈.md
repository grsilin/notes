# Activity 与 Activity 调用栈

## Activity

***Activity形态***

* Active / Running

  Activity处于最顶层，可见并与用户进行交互。

* Paused

  Activity失去焦点，被 一个新的非全屏的 Activity 或者一个透明的 Activity 旋转在栈顶时，Activity转化为Paused 形态。此时 Activity 仅仅失去了与用户交互的能力，所有状态、成员变量都被保留。

* Stopped

  Activity 被另一个 Activity 完全覆盖，进入 Stopped 状态，此时 Activity 不再可见，但是依然保持了所有状态信息和成员变量。

* Killed

  Activity 被系统回收或 Activity 从来没有创建过，Activity就会处于Killed 状态。

***生命周期***

![Alt text](./activity_lifecycle.png)

Activity 的稳定状态

* Resumed

  此时 Activity 处于栈顶，处理用户的交互。

* Paused

  Activity 不会接收用户的输入。

* Stopped

  Activity 仅在后台运行，不可见。

***Activity 的启动与销毁过程***

在系统调用 onCreate() 之后，就会马上 onStart()，然后继续调用onResume()以进入 `Resume`的状态，最后停留在 `resumed`状态，完成启动。系统会调用 onDestroy() 来结束一个 Activity 的声明周期，让它回到 Killed 状态。

* onCreate()：创建基本的UI元素。
* onPause()和onStop():清除 Activity的资源，避免浪费。
* onDestory():因为引用会在 Activity 销毁的时候销毁，而线程不会，所有清除开启的线程。

![Alt text](./basic-lifecycle.png)



***Activity 的暂停与恢复过程***

当栈顶的 Activity部分不可见后，就会导致 Activity 进入Pause 形态，此时就会调用`onPause()`方法，当结束阻塞后，就会调用`onResume()`方法来恢复到 Resume形态。

onPause()：释放系统资源，如 Camera 、sensor 、 receivers。

onResume()：需要重新初始化onPause()中释放的释放的资源。



***Activity 的停止过程***

栈顶的 Activity 部分不可见时，实际上有两种可能，从部分不可见到可见，也就是恢复过程，或是从部分不可见到完全不可见，也就是停止过程。系统在当前 Activity 不可见的时候，总会调用 onPause() 方法。



***Activity 重新创建的过程***

系统回收 Activity 时会将 Activity 状态通过 `onSaveInstanceState()`方法保存到 Bundle 对象中，在该方法中可以增加额外的键值对存入 Bundle 对象，以保存这些状态。当重新创建 Activity 时，保存的 Bundle 对象就会传递到 Activity 的 onRestoreInstanceState() 方法与 onCreate() 方法中。

## Android 任务栈

Android 系统通过栈结构来保存整个 App 的 Activity ，栈底的元素是整个任务栈发起者。

当一个 App 启动时，如果当前环境中不存在该 App 的任务栈，那么系统就会创建一个任务栈，此后，这个 App 所启动的 Activity 都在这个任务栈中被管理，这个栈也被 称为一个 Task ，即表示若干个 Activity 的集合。值得注意的是：一个 Task 中的Activity 可以来自不同的 App ，同一个 App 的Activity 也可能在不在一个 Task 中。

Activity 在当前栈结构中的位置，决定了Activity 的状态。正常情况下，当一个Activity启动了另一个 Activity 的时候，新启动的 Activity 就会置于任务栈的顶端，并处于活动状态，启动它的 Activity 虽然仍保留在任务栈中，但进入了停止状态，当顶部的 Activity 被关闭时，它又将恢复活动状态。通过设置 Activity 的启动模式，改变任务栈的管理方式。

### AndroidManifest 启动模式

* standard

  默认的启动模式，每次启动 Activity 都将创建新的实例，并置于栈顶。 

* singleTop

  如果需要启动的 Activity 当前处于栈顶，则不创建新的实例，而是直接引用这个Activity ，并调用该 Activity 的 `onNewIntent()`方法，如果不是则创建新的 Activity 。

* singleTask

  如果在当前栈内存在需要启动的 Activity 实例，将该 Activity 实例之上的所有的 Activity 销毁，从而使该 Activity 存在于栈顶，如果不存在则创建该 Activity 的实例。当从其它应用启动该 Activity 时，那么将创建一个新的任务栈。

  以 singleTask 模式启动的 Activity 已经在后台一个任务栈中，此时启动这个 Activity 将会使后台这个任务栈一起被切换到前台。

* singleInstance

  以 singleInstance 模式启动的 Activity 会出现在一个新的任务栈中，而且这个任务栈只有这一个 Activity，当另一个应用启动这个 Activity 时，不会重新创建 Activity 的实例。

关于singleTop 和 singleInstance 两种启动模式：

如果在个singleTop 或者 singleInstance 的 ActivityA 中通过 startActivityForResult() 方法来启动一个 ActivityB，那么系统将直接返回 Activity.RESULT_CANCELED 而不会等侍返回。因此在不同的 Task 之间传递数据需要通过 Intent 绑定数据来完成 。

### Intent Flag 启动模式

通过 Intent 启动 Activity 时，设置 Flag 来控制Activity 的启动模式。

常用的Flag有：

* Intent.FLAG_ACTIVITY_NEW_TASK

  使用一个新的 Task 来启动一个 Activity，但启动的每个 Activity 都将在一个新的 Task 中，通常用于从 Service （此时没有 Activity 栈）来启动一个 Activity。

* Intent.FLAG_ACTIVITY_SINGLE_TOP

  使用 singleTop 模式来启动一个 Activity

* Intent.FLAG_ACTIVITY_CLEAR_TOP

  使用 singleTask 模式来启动一个 Activity

* Intent.FLAG_ACTIVITY_NO_HISTORY

  使用这种模式启动 Activity，当该 Activity 启动其它 Activity 后，该 Activity 就消失了，不会保留在任务栈中。

## 清空任务栈

在 AndroidManifest 文件中使用以下几种属性来清理任务栈：

* clearTaskOnLaunch

  在每次返回该 Activity 时，都将该 Activity 之上的所有 Activity 清除，通过这个属性，可以让这个 Task 在每次初始化的时候，都只有这一个 Activity 。

* finishOnTaskLaunch

  通过这个属性，当离开这个 Activity 所处的 Task，那么用户再返回时，该 Activity 就会被 finish 掉。

* alwaysRetainTaskState

  Activity 不接受任何清理命令，一直保持当前 Task 状态。



## Activity 任务栈使用

合理使用任务栈