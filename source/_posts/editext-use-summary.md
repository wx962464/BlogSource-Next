title: EditText使用小结
date: 2016-05-12 00:52:43
tags: Android 
---
**来自：**   [Android梦想特工队](http://www.wxtlife.com/) 
**作者：**  [Aaron](http://www.wxtlife.com/about/) 
**主页：**  [http://www.wxtlife.com/](http://www.wxtlife.com/) 
**原文连接：** [http://www.wxtlife.com/2016/05/12/editext-use-summary/](http://www.wxtlife.com/2016/05/12/editext-use-summary/)


## EditText点击首次获得焦点后默认光标在最后的实现

这个本身是一个比较简单的问题了，大家一想都知道，设置`OnFocusChangeListener`方法，在hasFocus为true时调用`setSelection(int)`方法将光标移动到最后的位置。然后将事件绑定到EditText上即可，但是但是... 不起作用，不起作用，代码如下：

```java
editText.setOnFocusChangeListener(mFocusChangeListener);

private  View.OnFocusChangeListener mFocusChangeListener =  new View.OnFocusChangeListener()  {   
  @Override   
  public void onFocusChange(View v, boolean hasFocus) {         
      if(hasFocus) {        
         Log.v("Aaron","has focus ");   
         setEditTextCursorToLast();      
       } 
}};

private void setEditTextCursorToLast() {   
 editText.setSelection(editText.getText().toString().length());    
}
```
百思不得其解，搜索半天各种方法试过一轮后，怀疑是点击时候是先执行了onFocus，之后由于点击事件强制将光标又移至到点击的位置，导致不能设置的问题。那么尝试解决这个问题就是在focus的时候延迟下执行。。。把上面方法改为如下：
```java
private void setEditTextCursorToLast() {   
    edittext.post( new Runnable() {     
     @Override      
      public void run() {     
          mMailTitleEditText.setSelection(mMailTitleEditText.getText().toString().length());    
      }  
   });
}
```
其实也没有怎么延迟了。。仅仅是把执行的代码放在了一个runnable里面，然后结果就是这么神奇，结果起作用了。。

如果谁知道是什么原理请告知，不慎感激。