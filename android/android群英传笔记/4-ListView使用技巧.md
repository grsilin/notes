#ListView使用技巧

## 常用优化技巧

### ViewHolder提高效率

ViewHolder模式充分利用了ListView的缓存机制，避免了每次在用`getView()`的时候都通过`findViewById()`实例化控件。

首先在自定义Adapter中定义一个内部类`ViewHolder`，并将布局中的控件作为成员变量

```
public final class ViewHolder{
  public ImageView img;
  public TextView title;
}
```

在`getView()`方法中通过视图缓存机制来重用控件引用

```
pubic getView(int position,View convertView ViewGroup parent){
  ViewHolder hoder=null;
  if(convertView==null){
    holder=new ViewHolder();
    convertView=mInflate.inflate(R.layout.viewholder_item,null);
    holder.img=(ImageView)findViewById(R.id.imageView);
    holder.title=(TextView)findViewById(R.id.title);
    convertView.setTag(holder);
  }else{
    holder=(ViewHolder)convertView.getTag();
  }
  holder.img.setBackgroundResource(...);
  holder.title.setTest(...);
  return converView;
}
ViewHolder(){
  ...
}
```

### 设置项目间的分隔线

系统提供了`divider` 和`dividerHeight`控制LiveView的分隔线和它的高度。

```
android:divider="@android:color/darker_gray"
android:deviderHeight="10dp"
```

分隔线设置为透明

`android:diver="@null"`

### 隐藏ListView滚动条

设置`scrollbars`属性，控制滚动条状态

`android:scrollbars="none"`  	不显示滚动条

### 取消ListView的Item点击效果

`android:listSelector="#00000000"`

`android:selector="@android:coldor/transparent"`

### 设置ListView需要显示在第几项

`listView.setSelection(N);`

平滑移动：

```
mListView.smotthScrooBy(distance,duration);
mListView.smotthScrollByoffset(offset);
mListView.smotthScrollToPostion(index);
```

### 动态修改ListView

`mAdapter.nofityDatasetChanged();`

当修改传递给Adapter的映射之后，只需要通过调用Adapter的`notifyDataSetChanged()`方法，通知ListView更改数据源即可完成对ListView的动态修改。必须保证传递进Adaper的数据List是同一个List而不是其它对象

### 遍历ListView中所有的Item

通过`getChildAt()`来获取第i个子View

```
for(int i =0;i<mListView.getChildCount();i++){
  childView=mListView.getChildAt(i);
}
```

### 处理空ListView

通过`setEmtpyView()`给ListView设置一个在空数据下显示的默认提示

```
mListView.setEmptyView(findViewById(R.id.img_empty_tip));
```

### ListView滑动监听

**onTouchListener**

`onTouchListener`是View中的监听，通过监听`ACTION_DOWN`、`ACTION_MOVE`、`ACTION_UP`这三个事件发生时的坐标来判断用户的滑动方向。

```
mListView.setonTouchListener(new OnTouchListener(){
  public boolean onTouch(View v,MotionEvent event){
    case (event.getActions){
      case MotionEvent.ACTION_DOWN:
      	break;
      case MotionEvent.ACTION_MOVE:
      	break
      case MotionEvent.ACTION_UP;
      	break;
    }
    return false;
  }
});
```

**onScrollListener**

```java
mListView.setOnScrollListener(new OnScrollListener(){
  public void onScrollStateChanged(AbsListView view,int scrollState){
    switch(scrollState){
      case onScrollListener.SCROLL_STATE_IDLE:
      	//空闲状态，滑动停止时
      	break;
      case onScrollListener.SCROLL_STATE_TOUCH_SCROLL:
      	//正在滚动
      	break;
      case onScrollListener.SCROLL_STATE_FLING:
      	//惯性滑动状态
      	break;
    }
  }
  public void onScroll(	AbsListView view,
  					int firstVisibleItem,
  					int visibleItemCount,
  					int totlaItemcount
  	){
    //...
  }
});
```

OnScrollListener中有两个回调方法，`onScrollStateChanged()`和`onScroll()`

`onScrollStateChanged()`中的`scrollState`共有三种状态：

```
OnScrollListener.SCROLL_STATE_IDLE
//滚动停止时
OnScrollListener.SCROLL_STATE_TOUCH_SCROLL
//正在滚动时
OnScrollListener.SCROLL_STATE_FLING
//手指抛动时，当没有惯性滑动状态时不会有这一次调用
```

`onScroll()`会在ListView滚动时一直回调,其中参数的意义：

```
firstVisibleItem:当前参看见的第一个Item的ID（从0开始）
visibleItemCount:当前能看见的Item总数
totalItemCount:整个ListView的Item总数
```

通过这些参数，可以判断滚动方向，是否滚动到顶部（底部）等

```
if(firstVisibleItem+visibleItemCount==totalItemCount&&totlaItemCount>0){
  //滚动到最后一行
}
if(firstVisibleItem>lastVisivleItemPosition){
  //上滑
}
if(firstVisibleItem<lastVisibleItemPostion){
  //下滑
}
```

### ListView常用拓展

**具有弹性的ListView**

修改ListView中`overScrollBy`方法，使ListView具有弹性效果

```
protected boolean onScrollBy(int deltaX,int deltaY,
							int scrollX,int scrollY
							int scrollRangeX,int scrollRangeY,
							int maxOverScrollX,int maxOverScrollY
							boolean isTouchEvent){
							//
							DisplayMetrics metircs=mContext.getResource.getDisplayMetrics();
							float density=metrics.density;
							mMaxOverDistance=int (density*mMaxOverDistance);
    	return super.overScrollBy(deltaX,deltaY,
    							scrollX,scrollY,
    							maxOverScrollX,mMaxOverDistance,
    							isTouchEvent);                         
	}
```

**自动显示、隐藏的ListView**

```
//给ListView增加一个HeadView，避免第一个Item被TollBar遮挡
View Header =new View(this);
header.setLayoutParams(new AbsListView.LayoutParams(
AbsListView.LayoutParams.MATCH_PARAENT,(int)getResource.getDemension(R.dimen.abc_action_bar_default_default_height_material)));//获取系统Actionbar的高度，并设置给HeaderView
mListView.addHeader(header);
```

定义一个变量`mTouchSlop`用来获取系统认为的最低滑动距离，即超过这个距离的移动，系统就将其定义为滑动状态。

```
mTouchSlop=ViewConfiguration.get(this).getScaledTouchSlop();
```

判断滑动事件

```
View.OnTouchListener listener=new View.OnTouchListener(){
  public void onTouch(View v,MotionEvent event){
    
  }
}
```

