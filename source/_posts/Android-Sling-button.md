title: "滑动按钮（Sliding Button）实现"  
date: 2015-04-06 12:12:54  
categories: Android  
tags: [Android UI] 
---

滑动控件在我们的应用中还是比较常见的，比如锁屏的那个滑动效果，偶尔也会使用到滑动清除功能等，滑动到末端触发的一些事情。下面就说说这个要怎么实现它呢，看完你会觉得原来如此简单，代码也不多。  
首先来看看效果图：   
![SlidingButton](/img/sliding_button.jpg)  
首先得有个滑块，然后要限定滑块的滑动位置，这两个元素就够我们实现一个滑动的效果了，那么滑动要怎么移动呢？这就是本文的核心点了。当然采用自定义view的方式来实现啦。  
<!-- more -->   
要实现view的移动，那么我们就得知道view到底移动了多少，向左还是向右，看到这，那就必须监听view的onTouch事件了，所以ontouch 事件的处理才是核心的核心。ontouch事件可以告诉view偏移多少，然后将view移动多少就可以实时的变化了，view的移动采用设置`LayoutParams的leftMargin`来实现。下面看看核心代码是如何写的。
    我们继承ImageButton来实现，
    ` class SlidingButton extends ImageButton`
        
```java
 //我们继承ImageButton，然后重写Ontouch事件。
  @Override
    public boolean onTouchEvent(MotionEvent event) {

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                eventX = (int)event.getX();
//              handlerMoveEvent(event);
                break;
            case MotionEvent.ACTION_MOVE:
                handlerMoveEvent(event);
                break;
            case MotionEvent.ACTION_UP:
                handlerUpEvent(event);
                break;
            default:
                break;
        }
        return super.onTouchEvent(event);
    }
  //处理移动过程中的事件，即移动滑块。 
  private void handlerMoveEvent(MotionEvent event) {
        mParentWidth = ((ViewGroup) this.getParent()).getWidth();
        int x = (int) event.getX();
        Log.v("Aaron","touch x = " + x + " getLeft = " +getLeft() + " x dist = " + (x - eventX) );
        int marginLeft = getLeft()+(x - eventX);
        Log.v("Aaron","margin left = " + marginLeft);
        //不做中间的值的判断，而使用最大和最小的判断，是因为，当快速滑动到时候两个evevtX的距离会很远，会瞬间超过最大值，
        //此时则不会设置margin了，也就会出现滑不到最右端的bug。所以必须这样处理。
        if(marginLeft < mOriginMarginleft ) {
            marginLeft = mOriginMarginleft;
        } else if(marginLeft > mParentWidth - getWidth()) {
            marginLeft = mParentWidth - getWidth();
        }
        LinearLayout.LayoutParams lp = (LinearLayout.LayoutParams)getLayoutParams();
        lp.leftMargin =marginLeft;
        setLayoutParams(lp);
        //为什么不能使用下面这句，按照常规的逻辑应该为eventX重新赋值的，但是如果加上上面那句就会发现有问题了，弄清楚这个也就弄处理它是怎么滑动的了。
        // x每次的值是相对于当前view上的，move操作会让x1变化为x2，然后变化的量（x2-x1）将其设置为view的margin，这时view位置变化了，
        // view变化后x2的值实际相当于未移动view的x1的值，然后在进行move，又发生偏移，回到上面的逻辑，也就是说偏移量margin相当于重置了x的值。
        // 所以不需要重新赋值，这也是view变化的巧妙之处。
//      eventX = x ;

    }   
    
     private void handlerUpEvent(MotionEvent event) {
        mParentWidth = ((ViewGroup) this.getParent()).getWidth();
        if (this.getLeft() + getWidth() > mParentWidth - getWidth() / 2) {
            //达到最右端，通知已经可以触发相应的事件了。
            fireSlidingSuccessListener();
            startAnimation(50);
        } else {
            //没有到最右端，则将其自动滑动到原始的位置。
            startAnimation(200);
        }
    }
    //外部设置回调的方法。
    public void setSlidingListener(ISlidingListener listener) {
        mSlidingListener = listener;
    }
    
    private void fireSlidingSuccessListener() {
        if(mSlidingListener != null){
            mSlidingListener.slidingSuccess();
        }
    }
    
    //处理事件成功的回调。
   public interface  ISlidingListener {
        public void slidingSuccess();
    }
    
```
Demo 下载地址：[http://download.csdn.net/detail/wx_962464/8567499](http://download.csdn.net/detail/wx_962464/8567499)

这样核心的都在代码里面已经注释了，不懂的可以留言。
