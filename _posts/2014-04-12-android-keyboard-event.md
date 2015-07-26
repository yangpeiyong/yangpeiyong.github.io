---
layout:     post
title:      在Android中监听键盘出现和隐藏事件
date:       2014-04-12 12:32:18
summary:    Android 没有键盘消失和出现的监听接口，那么我需要监听键盘事件的时候，该如何做呢？
categories: Android
---

##一、缘起

     Android 没有键盘消失和出现的监听接口

     有的时候需要根据键盘消失和出现来更改UI布局

##二、实现方案

1、自定义layout
{% highlight java linenos %}
public class YPYRelativeLayout extends RelativeLayout {  
        @Override
        protected void onLayout(boolean changed, int l, int t, int r, int b) {  
               super.onLayout(changed, l, t, r, b);  
             //在这里通过检测layout高度(b-t)的变化来判断键盘是否出现
       }  
}
{% endhighlight %}
这种方案需要在manifest文件中加入：

android:windowSoftInputMode = “adjustResize|stateHidden”



这样layout才会随键盘的出现和消失来调整大小

存在问题：由于Android系统的BUG，在全屏模式下，这种方式无法使用

2、监听addOnGlobalLayoutListener
{% highlight java %}
findViewById(R.id.main).getViewTreeObserver(). addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {  

            @Override
            public void onGlobalLayout() {  
                // TODO Auto-generated method stub  
                Rect r = new Rect();  
                mRelativeLayout.getWindowVisibleDisplayFrame(r);  
                int screenHeight = mRelativeLayout .getRootView().getHeight();  
                int heightDifference = screenHeight – (r.bottom – r. top);  
                boolean visible = heightDifference > 100;  
            }  
        });  
{%endhighlight}
这种方式在所有的情况下都可以使用。

[Demo 下载](http://pan.baidu.com/s/1kT7H8ZP)
