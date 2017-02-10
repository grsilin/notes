# Android性能优化

内容列表：

* 布局优化
* 内存优化
* 分析优化工具

## 布局优化

***Andorid UI渲染***

在android中，系统通过 VSYNC 信号触发对 UI 的渲染、重绘，其间隔时间是16ms，即每秒的帧数为60，如果每次渲染都在16ms内完成，就可以得到流畅的画面。但如果绘制任务没有在16ms内完成，当前重绘的帧被未完成的任务阻塞，就会造成丢帧，系统要等到下次 VSYNC 信号才进行重绘，此时就会有画面卡顿的现象出现。

***避免 Overdraw***

过度绘制会浪费很多CPU、GPU资源，例如系统默认会绘制Activity的背景，如果再给布局绘制了重叠的背景，那么默认 Activity 的背景就属于无效的过度绘制——Overdraw。

***优化布局层级***

在 Android 中，系统对 View 进行测量、布局和绘制时，都是通过对 View 数的遍历来进行操作的，如果一个 View树太高，就会严重影响测量、布局和绘制的速度，因此优化布局的第一个方法就是降低 View 树的高度（不超过10层）。

***避免嵌套过多无用布局***

嵌套的布局会让 View 树的高度变得越来越高，因此在布局时，需要根据自身布局的特点来选择不同的 Layout 组件，从而避免通过某一种 Layout 组件来实现功能时的局限性，从而造成嵌套过多的情况发生。

**使用<include>标签重用Layout**

<include>标签用来引用一个共通的UI布局。

```xml
<RelativeLayout mxlns:andorid="http://schemas.android.com/apk/android"
                >
	<include layout="@layout/common_ui"
             android:layout_alignParentBototn="true"
             andorid:layout_width="match_parent"
             andorid:layout_width="match_parent"
             ></include>
</RelativeLayout>
```

注：在<include>标签中同样可以使用 Layout 组件的一些属性来控制引用的布局，但如果要覆盖原布局中的 layout_XXXX 的属性，就须在 <include> 中同时指定 layout_width 和 layout_height 属性。

**View的延迟加载**

使用 <ViewStub> 标签来实现对一个 View 的引用并实现延迟加载。<ViewStub>是一个轻量级的组件，不仅不可视，而且大小为 0。

创建一个布局，这个布局在初始化时不显示，在某些情况下才显示出来。

```xml
<?xml version="1.0" encoding ="utf-8"?>
<LinearLayout xmlns:andorid="http://schemas.android.com/res/apk/android"
              andorid:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
 	<TextView
              android:id="@+id/tv"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:text="not ofen use layout"
              android:textSize="30sp"
              ></TextView>
</LinearLayout>
```

在主布局中的<ViewStub>中的 layout 属性来引用上面的布局：

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="wrap_content"
                android:layout_height="match_parent"
                >
	<ViewStub
             andorid:layout="@layout/layout_not_ofen_use"
             android:layout_width="match_parent"
             andoird:layout_height="wrap_content"
             ></ViewStub>
</RelativeLayout>
```

重新加载显示布局：

```java
mViewStub = (ViewStub) findViewById(R.id.layout_not_ofen_use);
//方法1
mViewStub.setVisibility(View.VISIBLE);
//方法2
View mInflateView = mViewSub.inflate();
```

方法2可以返回布局的引用，通过这个引用可以获取到布局中的控件。

一旦<ViewSub>被设置为可见或被 inflate 了，<ViewStub>就不存在了，取而代之的是被 inflate 的 Layout ，并将这个 Layout  的 ID 重新设置为 <ViewStub> 中通过android:infaltedId 属性指定的 ID。

<ViewStub> 的作用是只有在显示时才去渲染整个布局，而不是在初始化时添加到布局树中，从而提高加载的效率。

***Hierachy Viewer***



## 内存优化

### 什么 是内存

由于 Android 应用的沙箱机制，每个应用所分配的内存大小是有限度的，内存太低就会触发 LMK ——Low Memory Killer 机制。

通常情况下我们所说 的内存主要包括以下几个部分：

* 寄存器（Registers）

  速度最快的存储场所，在处理器内部，程序无法控制

* 栈（Stack）

  存放基本类型和对象的引用

* 堆（Heap）

  存放由 new 创建的对象和数组。在堆中分配的内存，由Java虚拟机的垃圾回收器（GC）来管理。

* 静态存储区域（Static Field）

  指在固定的位置存放应用程序运行时一直存在的数据。Java在内存中专门划分了一个静态存储区域来管理一些特殊的数据变量如静态的数据变量。

* 常量池

  JVM必须为每个被装载的维护一个常量池。常量池就是该类型所用到的常量的一个有序集合，包括直接常量（基本类型，String）和对其它类型、字段和方法的符号引用。

  ```java
  //获取堆的大小
  ActivityManager manager = (ActivityManager)getSystemService(Context.ACTIVITY_SERVICE);
  int heapSize = manager.getLargeMemoryClass();
  ```

### 获取 Android 系统内存信息

***Progress Stats***

Progress Stats 是 KK 上新增的一个系统内存监视服务，可以通过“Setting-developer options-Progress Stats” 来开启该功能的界面，或使用"Dumpsys"命令来获取这些信息。

```shell
adb shell dumpsys procstats
```

**Meminfo**

Meminfo 也是系统上一个非常重要的内存监视工具，在"Setting -Apps-Running"中打开。

命令方式

```shell
adb shell dumpsys meminfo
```

### 内存回收

Java 创建了垃圾收集器线程（Garbage Collection Thread）来自动进行资源的管理，但是在程序中无法控制何时进行回收，可能存在部分对象没有被回收，从而造成内存泄漏。

### 内存优化实例

***Bitmap优化***

* 使用适当分辨率的大小的图片

  Android 系统在做资源适配时会对不同分辨率文件平下的图片进行缩放来适配相应的分辨率，如果图片分辨率不匹配或者图片分辨率太高，就会导致系统消耗更多的内存资源，同时在适当的时候应该显示合适大小的图片，如在图片列表界面可以使用图片的缩略图（thumbnails），在对图像要求不高的地方尽量降低图片的精度。

* 及时回收内存

  使用完 Bitmap 后，一定要及时使用 bitmap.recycle()方法释放内存资源。

* 使用图片缓存

* 通过内存缓存（LruCache）和硬盘缓存（DiskLruCache）可以更好的使用 Bitmap。

***代码优化***

* 对常量使用 static 修饰符
* 使用静态方法，静态方法会比普通方法提高15%左右的访问速度。
* 减少不必要的成员变量，如果一个变量可以定义为局部变量，内里不要定义为成员变量。
* 减少不必要的对象，使用基础类型会比使用对你更加节省资源，同时应该避免频繁创建短作用域的变量。
* 尽量不要使用枚举、少用迭代器
* 对 Cursor、 Receiver、Sensor、File 等对象，要非常注意它们的创建、回收与注册、解注册。
* 避免使用 IOC 框架，IOC 框架通常使用注解、反射实现，可能带来性能的下降。
* 使用 RenderScript 、 OpenGL 来进行非常复杂的绘图操作。
* 使用 SurfaceView 来替代View 进行大量、频繁的绘图操作
* 尽量使用视图缓存，而不是每次都执行 inflate()方法解析视图。

### Lint 工具

Android Lint工具是Android 中集成的一个 Android 代码提示工具，它可以给你的布局、代码提供非常强大的帮助。

### Memory  Monitor 工具

Memory Monitor 是Android Studio 自带的一个内存监视工具，用于进行内存实时分析。

## 使用 TraceView 优化 App 性能

一、生成日志

* 通过代码生成精确范围的 TraceView 日志

  使用Debug类开启TraceView监听。通过调用Debug.startMethodTracing()方法可以开启监听，Debug.stopMethodTracing()方法结束监听，使用这两个方法包围要监听的代码块，TraceView 将会监听这些代码，并将生成的日志保存到"/sdcard/dmtrace.trace"目录下。

  （需要添加外部存储写权限

  `<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE">`）

  当执行完监听的内容后，通过ADB命令将日志文件导出到本地

  ```shell
  adb pull /sdcard/dmtrace.trace /local/LOG/
  ```

* 通过 Android Device Monitor 生成 Trace View 日志

  在 Android Device Monitor 工具中，选择要调试的进程，点击工具栏中的"start method profiling"，选择监听模式，进行监听，有两种监听方式可供选择：

  **整体监听**

  跟踪每个方法执行的全部过程

  **抽样监听**

  按照指定频率来进行抽样调查

二、打开TraceView日志

​	使用SDK工具中的“sdk\tools\traceview.bat”或 ADM 工具中，在“File” 菜单下选择“Open File...” 选项打开 TraceView 日志文件。

三、分析 TraceView日志

TraceView 的分析界面分为两个部分，下面是用于显示方法执行时间的时间轴区域，下面是显示详细信息的 profile 区域。

***时间轴区域***

时间轴区域显示了不同的时间段内的执行情况，在时间轴中，每一行都代表了一个独立的线程。不同的色块代表了不同的执行方法，色块的长度代表了方法执行的时间。

***Profile区域***

Profile 区域显示所选择色块所代表的方法在该色块所处时间段内的性能分析。

主要包括以下内容：

* Incl CPU Time ——某方法占用的 CPU时间。
* Excl CPU Time —— 某方法本身（不包括子方法）占用的 CPU 时间。
* Incl Real Time —— 某方法真正执行的时间。
* Excl Teal  Time ——某方法本身（不包括子方法）真正执行的时间。
* Calls+RecurCalls——调用次数 + 递归回调的次数

每个时间都包含两列，一个是实际的时间，另一个是百分比。

通常从Incl CPU Time 和 Calls + RecurCalls 开始进行分析，对占用时间长的方法进行重点分析，如果占用时间长且 Calls + RecurCalls 次数少，那么就可以列为怀疑对象了。

## 使用MAT 工具分析App内存状态

一、生成 HPROF 文件

首先打开 Android Device Monitor 工具，选择要监听的线程，并点击菜单栏中的“Update Heap” 按钮，在 Heap 标签中，点击 “Cause GC” 按钮，就会显示当前的内存状态。（不停的点击“Cause GC” 按钮时，如果 “data Object” 一栏中的“Total Size”有明显变化，就代表可以存在内存泄漏。）点击菜单栏中的“Dump HPROF File”按钮，等待几秒钟，系统就会生成一个".hprof"文件，默认名称为"packakgename.hprof"。

使用SDK目录下的“platform-tools/hprof-conv”工具进行转换：

```shell
hprof-conv com.example.heap.hprof
```

二、分析 HPROF 文件

打开 MAT 工具，选择 “Open  a heap Dump” 选项，等待文件导入后，将显示分析结果。

主要项目：

* Histogram

  Histogram 直方图，用于显示内存中每个对象的数量、大小和名称。

* Dominator Tree

  Domainator Tree（支配树）将内存中的对象按照大小进行排序，并显示对象之间的引用结构。

## 使用Dumpsys 命令分析系统状态

使用`dumpsys`命令可以列出 Android 系统相关的信息和服务状态。

```shell
adb shell dumpsys arguments
```

常用的Dumpsys 参数



| 参数        | 意义                |
| --------- | ----------------- |
| activity  | 显示所有的Activity栈的信息 |
| meminfo   | 内存信息              |
| battery   | 电池信息              |
| package   | 包信息               |
| wifi      | wifi 信息           |
| alarm     | alarm 信息          |
| procstats | 显示内存状态            |

