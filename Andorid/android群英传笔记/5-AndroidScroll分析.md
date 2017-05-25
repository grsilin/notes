
Android Scroll 分析
---
## 如何使View滑动

### Android坐标系
  屏幕最左上角的顶点作为Android坐标系的原点，这个原点向右是X轴正方向，从这个点向下是Y轴正方向。
通过getlocationOnScreen来获取Android坐标系中点的位置，或有触控事件中使用`getRawX()`,`getRawY()`
来获取Android坐标系中的坐标。

### 视图坐标系
  以父视图左上角为原点，在触控事件中通过`getX()`,`getY()`获取视图坐标系中的坐标。

### 触控事件
  触控事件的类型

    ACTION_DOWN
    //单点触摸按下
    ACTION_UP
    //单点触摸抬起
    ACTION_MOVE
    //触摸点移动
    ACTION_CANCEL
    //触摸动作取消
    ACTION_OUTSIDE
    //触摸动作超出边界
    ACTION_POINTER_DOWN
    //多点触摸按下
    ACTION_POINTER_UP
    //多点触摸离开

### 获取坐标的方法
  view获取坐标的方法
  `getTop()`  获取View本身顶边到父布局顶边的距离
  `getLeft()`
  `getRight()`
  `getBotton()`
  MotionEvent获取坐标的方法
  `getX()`
  `getY()`
  `getRawX()`
  `getRawY()`
## 实现滑动的七种方式
*   一、layout方法
    在`onTouchEvent()`方法中计算偏移量，并调用`layout()`方法，改变View的布局，从而达到移动View的效果。

*   二、offsetLeftAndRight()与offsetTopAndButton()
        在`onTouchEvent()`中获取坐标计算偏移量,调用offsetLeftAndRight(dx),offsetTopAndButton(dy)使View移

*   三、LayoutParams

      LayoutParams保存了一个View的布局参数，因此可以在程序中通过改变LayoutParams来动态的修改一个布局的位置参数，从而达到改变View位置的效果。通过`getLayoutParams()`来获取一个View的LayoutParams，计算偏移量并改变LayoutParams中的值，然后通过`setLayoutParams()`来应用修改后的LayoutParams。（LayoutParams可以是对应父布局的LayoutParams也可以是ViewGroup的MarginLayoutParams）

*   四、scrollTo()与scrollBy()
    scrollTo()中的参数是移动位置，scrollBy()的参数是偏移量。
    当参数为正数时，向坐标轴向负方向移动。

*   五、Scroller


```java
//初始化scroller
mScroller = new Scroller(context);
//重写computeScroll()
@Override 
public void computeScroll(){
  super.computeScroll();
  if(mScroller.computeScrollOffset()){
    ((View)getParent()).scrollTo(
    	mScroller.getCurrentX(),mScroller.getCurrentY);
    invalidate();//invalidate()-->draw()()-->computeScroll()
  }
}
```

`startScroll()`开启模拟过程

```java
public void startScroll(int startX,int startY,
                       int dx,int dy,int duration)
 public void startScroll(int startX,int starty,int dx,int dy)
```

* 六、属性动画
* 七、ViewDragerHelper

1、初始化ViewDragHelper

```java
mViewDragerHelper = ViewDragHelper.create(this,callback);
```

ViewdragHelper通过定义在一个ViewGroup内部，通过静态工厂进行初始化。它的第一个参数是要监听的View，通常是一个ViewGroup，第二个参数是一个Callback回调（核心逻辑）。

2、拦截事件

重写事件拦截方法，将事件传递给ViewDraghelper进行处理。

```java
@Override
public boolean onInterceptTouchEvent(MotionEvent ev){
  retrun mViewDragHelper.shouldInterceptTouchEvent(ev);
}
@Ovrride
public boolean onTouchEvent(MotionEvent event){
  mViewDragHelper.processTouchEvent(event);
  return true;
}
```

3、处理computeScroll()

```java
@Override
public void computeScroll(){
  if(mViewDragHelper.continueSettling(true)){
    ViewCompat.postInvalidateOnAnimation(this);
  }
}
```

4、处理Callback

```java
private ViewDragHelper.Callback callback = new ViewDragHelper.Callback(){
  @Override
  public boolean tryCaptureView(View child , int pointerId){
    return mMainView == child;//只有Mainview可以被移动
  }
}
```

在`cryCaptureView()`中指定哪个子View可以被移动。

滑动：

```java
@Override
public int clampViewPositionVertical(View child,int top,int dy){
  //top代表在垂直方向上child移动的距离，dy表示比较前一次的增量
  return top;
}
@Override
public int clampViewPositionHorizontal(View child,int letf,int dx{
  return left;
}
```

`clampViewPositionVertical()`和`clampViewPostionHorizontal()`分别对应垂直方向和水平方向上的滑动。

手指离开屏幕：

重写`onViewReleased()`方法

```java
@Override
public void onViewRelease(View releasedChild,
                          float xvel,float yvel)
{
	super.onViewReleased(releasedChild,xvel,yvel);
    if(mMainView.getLeft()<500){
      //关闭菜单
      //相当于Scroller的startScroll方法
      mViewDragHelper.smotthSlieViewTo(mMainView,0,0);
      ViewCompat.postInvalidateONAnimation(DragViewGroup.this);
    }else{
      mViewDragHelper.smotthSlideViewTo(mMainView,300,0);
      ViewCompat.postInvalidateOnAnimation(DragViewGroup.this);
    }
}
```

`Callback`中可以监听的事件：

* onViewCaptured()

用户触摸到View后回调

* onViewDragStateChanged

这个事件在拖拽姿态改变时回调，比如idle，dragging

* onViewPositionChanged()

这个事件在位置改变时回调，常用于滑动时更改scale进行缩放等效果