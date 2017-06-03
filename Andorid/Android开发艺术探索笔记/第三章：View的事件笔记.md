### View 滑动的三种方式
  1.  通过scrollTo ()/scrollBy()
  2.  通过动画
  3.  改变布局参数
  优劣分析：
    scrollTo/By移动的是View中内容，而不能移支View本身的位置
    动画可以方便的实现移动，但3.0下系统属性动画不支持View实际位置的移动。
    改变布局参数可以改变view的实际位置，但使用过程较繁琐。  

### 弹性滑动

### 事件分发机制
  事件的分发过程就是对MotionEvent事件的分发过程。当一个MotionEvent产生以后，系统需要把这个事件传递给一个具体的View，这个过程就是分发的过程。
  事件分发主要由三个主要的方法来完成：
  > dispatchTouchEvent(MotionEvent ev)

  用于进行事件的分发，如果事件能够传递给当前的View，那么此方法一定被调用，返回结果受当前View的onTouchEvent和下级View的dispatchTouchEvent方法影响，表示是否消耗当前事件。
  > onInterceptTouchEvent(MotionEvent ev)

  在dispatchTouchEvent内部调用，用来判断是否拦截某个事件，如果当前View拦截了某个事件，那么在同一个事件序列中，此方法不会被再次调用，返回结果表示是否拦截当前事件。
  > onTouchEvent

  在dispatchTouchEvent方法中调用，用来处理单击事件，返回结果表示是否消耗当前事件，如果不消耗，则在同一个事件序列中当前View无法再次接收到事件。

结论：
<ol>
  <li>同一个事件序列是指从手指触屏到手机离开所产生的一系列事件，以`down`事件开始，中间有不定数目的`move`事件，最后以`up`事件结束。
  <li>正常情况下，一个事件序列只能被一个View所拦截且消耗。因为一个View一旦拦截了某个事件，那么同一个事件序列内的事件都会被传递给它处理,并且它的`onInterceptTouchEvent`不再会被调用。
  <li>某个view一旦开始处理事件，如果它不消耗`action down`事件（在`onTouchEvent`中返回false）,那么同一事件序列中的其它事件都不会交给它处理，并且父元素的`onTouchEvent`事件将会被调用。
  <li>如果View不消耗除`action down`以外的其它事件，那这个点击事件将会消失，此时父元素的`onTouchEvent`方法不会被调用，并且View可以持续收到事件序列，这些没有被消耗的事件最终由`Activity`处理。
  <li>ViewGroup默认不拦截任何事件。即在`onInterceptTouchEvent`默认返回false
  <li>View没有`onInterceptTouchEvent`方法，一旦有事件传递给它，它的`onTouchEvent`将会被执行。
  <li>View的`onTouchEvent`方法默认返回true,除非不可点击的,即`clickable`和`longclickable`均为false。
  <li>view的`enable`属性不影响`onTouchEvent`的返回值。
  <li>onClick会发生的前提是View是可点击的，并且它收到了`UP`和`DOWN`事件。
  <li>事件传递的过程是由外向内的，即事件总是先传递给父元素，再由父元素传弟给子，通过`requestDisallowInterceptTouchEvent`方法在子view中干扰父元素的事件分发过程(在`down`事件中会调用`resetTouchState`清除子view的设置)
<ol>
### View的滑动冲突
  常见冲突场景：
    1、外部滑动方向和内部滑动方向不一致
    2、外部滑动方向和内部滑动方向一致
    3、前两种情况的嵌套
  滑动冲突的解决方式：
    外部拦截法：重写父容器的`onInterceptTouchEvent`方法，由父容器决定是否拦截事件。
    内部拦截法：重写子元素的`dispatchTouchEvent`方法，配合`requestDisallowInterceptTouchEvent`来实现。父容器拦截除`ACTION_DOWN`以外的所有事件，在子元素中将不需要的事件通过`requestDisallowInterceptTouchEvent`交由父容器拦截。
