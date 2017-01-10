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
  * layout方法
    在`onTouchEvent()`方法中计算偏移量，并调用`layout()`方法，改变View的布局，从而达到移动View的效果。
  * offsetLeftAndRight()与offsetTopAndButton()
    在`onTouchEvent()`中获取坐标计算偏移量,调用offsetLeftAndRight(dx),offsetTopAndButton(dy)使View移动
  * LayoutParams
    LayoutParams保存了一个View的布局参数，因此可以在程序中通过改变LayoutParams来动态的修改一个布局的位置参数，从而达到改变View位置的效果。通过`getLayoutParams()`来获取一个View的LayoutParams，计算偏移量并改变LayoutParams中的值，然后通过`setLayoutParams()`来应用修改后的LayoutParams。（LayoutParams可以是对应父布局的LayoutParams也可以是ViewGroup的MarginLayoutParams）
  * scrollTo()与scrollBy()
    scrollTo()中的参数是移动位置，scrollBy()的参数是偏移量。
    当参数为正数时，向坐标轴
