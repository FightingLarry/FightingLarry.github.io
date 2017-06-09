---
layout: post
category: Android
title:  "Android View源码解析"
tags: [Android,View]
---

**一、官方文档**

​     先是看了一下官方的文档， [地址是http://developer.](http://developer.android.com/guide/topics/ui/how-android-draws.html)[Android](http://lib.csdn.net/base/android).com/guide/topics/ui/how-android-draws.html，它大体讲解了View的绘制流程。在此翻译一下，方便大家阅读。

​     当一个Activity接收焦点,它将被要求画出它的布局。Android框架将处理画图,但Activity必须提供根节点布局的层次结构。

​     从图的根节点开始布局（绘制），然后开始测量和绘制布局树。 通过遍历树和渲染来绘制每个视图,相交无效区域。反过来,每个视图组负责请求它的每个孩子绘制(draw() method)和每个视图负责绘画本身。因为树是遍历顺序,这意味着父母将被先绘制，孩子视图后被绘制。

​     注意：该框架将不会画视图中所没有的无效区域,也会注意绘图视图的背景。你能强迫一个视图来画，通过使用 invalidate()方法。

​     绘制视图（View）包含两个过程，一个是measure，一个是 layout。measure的过程是使用measure(int, int)方法，她也是自顶向下遍历树。在递归的过程中，每一个视图（View）将尺寸规格放入栈中，在measure过程的最后，每个视图存储了它的尺寸。layout的过程与measure差不多，先是调用layout(int, int, int, int)方法，然后自上而下遍历。在这个过程中，每个父视图（paraent）负责定位所有的孩子的位置，这个位置的数值就是由measure过程计算出来的。

​     当一个视图(View)的的measure()返回，它的`getMeasuredWidth()` 和 `getMeasuredHeight()` 的值 就必须被设置了，以及所有这些视图的孩子节点。一个视图的宽度和高度值必须在父视图约束范围之内，这可以保证最后的 measure 都通过,所有的父母都接受所有孩子的测量。父视图可以调用measure()方法不止一次在它的孩子上。

​     在measure过程中，使用了两个类来传递尺寸大小。一个是[ViewGroup.LayoutParams](http://developer.android.com/reference/android/view/ViewGroup.LayoutParams.html)，孩子视图使用这个类并告诉他们的父视图他们应该怎样被测量和放置。一个是基本的LayoutParams，用来描述视图的高度和宽度。它的尺寸可以有三种表示方法：1、具体数值 2、FILL_PARENT 3、WRAP_CONTENT

​     对于不同的ViewGroup的子类，有着各自不同的LayoutParams。例如,RelativeLayout有它自己的LayoutParams的子类,其中包括子视图的能力水平和垂直中心。

  MeasureSpecs用于推动需求下树的父母和孩子。	一个MeasureSpec可以在三种模式:

  UNSPECIFIED:

  EXACTLY:具体

  AT_MOST:

 

**二、基本概念以及流程**

​     可以看到，官方的文档只是泛泛的说了一下，没有具体详细的说明整个流程，然后在网上找了一些资料，结合官方文档来详细说了下，对于理解这个部分，效果可能会更好。在正式介绍之前，还是需要理解与UI相关的基本概念。

**Activity**：Activity包含一个Window，该Window在Activity的attach方法中通过调用PolicyManager.makeNewWindo创建；

**View**：最基本的UI组件，表示屏幕上的一个矩形区域；

**DecorView**：是Window中View的RootView，设置窗口属性；

**Window**：表示顶层窗口，管理界面的显示和事件的响应；每个Activity 均会创建一个 PhoneWindow对象，是Activity和整个View系统交互的接口

**WindowManager**：一个interface，继承自ViewManager。所在应用进程的窗口管理器；有一个implementation WindowManagerImpl；主要用来管理窗口的一些状态、属性、view增加、删除、更新、窗口顺序、消息收集和处理等。

**ViewRoot：**通过IWindowSession接口与全局窗口管理器进行交互：界面控制和消息响应；

**ActivityThread：**应用程序的主线程，其中会创建关联当前Activity与Window；创建WIndowManager实现类实例，把当前DecoView加入到WindowManager；

​     Android UI的[架构](http://lib.csdn.net/base/architecture)组成如图：

​                          ![img](http://img.my.csdn.net/uploads/201304/22/1366614231_7755.jpg)

​    上述架构很清晰的呈现了Activity、Window、DecorView（及其组成）、ViewRoot和WMS之间的关系，通过源码简单理了下从启动Activity到创建View的过程，大致如下：

​                       ![img](http://img.my.csdn.net/uploads/201304/22/1366614546_1278.jpg)          

​     在上图中，performLaunchActivity函数是关键函数，除了新建被调用的Activity实例外，还负责确保Activity所在的应用程序启动、读取manifest中关于此activity设置的主题信息以及上图中对“6.onCreate”调用也是通过对mInstrumentation.callActivityOnCreate来实现的。图中的“8. mContentParent.addView”其实就是架构图中phoneWindow内DecorView里面的ContentViews，该对象是一个ViewGroup类实例。在调用AddView之后，最终就会触发ViewRoot中的scheduleTraversals这个异步函数，从而进入ViewRoot的performTraversals函数，在performTraversals函数中就启动了View的绘制流程。performTraversals函数在2.3.5版本源码中就有近六百行的代码，跟我们绘制view相关的可以抽象成如下的简单流程图：

​                        ![img](http://img.my.csdn.net/uploads/201304/22/1366614772_1046.png)

​    上述流程主要调用了View的measure、layout和draw三个函数。

​    invalidate主要给需要重绘的视图添加DIRTY标记，并通过和父视图的矩形运算求得真正需要绘制的区域，并保存在ViewRoot中的mDirty变量中，最后调用scheduleTraversals发起重绘请求，scheduleTraversals会发送一个异步消息，最终调用performTraversals()执行重绘，。该函数可以由应用程序调用，或者由系统函数间接调用，例如setEnable(), setSelected(), setVisiblity()都会间接调用到invalidate()来请求View树重绘，更新View树的显示。注：requestLayout()和requestFocus()函数也会引起视图重绘。

​    invalidate()最后会发起一个View树遍历的请求，并通过执行performTraersal()来响应该请求，performTraersal()正是对View树进行遍历和绘制的核心函数，内部的主体逻辑是判断是否需要重新测量视图大小（measure），是否需要重新布局（layout），是否重新需要绘制（draw）。measure过程是遍历的前提，只有measure后才能进行布局（layout）和绘制（draw），因为在layout的过程中需要用到measure过程中计算得到的每个View的测量大小，而draw过程需要layout确定每个view的位置才能进行绘制。下面我们主要来探讨一下measure的主要过程，相对与layout和draw，measure过程理解起来比较困难。

​    我们在编写layout的xml文件时会碰到layout_width和layout_height两个属性，对于这两个属性我们有三种选择：赋值成具体的数值，match_parent或者wrap_content，而measure过程就是用来处理match_parent或者wrap_content，假如layout中规定所有View的layout_width和layout_height必须赋值成具体的数值，那么measure其实是没有必要的，但是google在设计Android的时候考虑加入match_parent或者wrap_content肯定是有原因的，它们会使得布局更加灵活。

​    首先我们来看几个关键的函数和参数：

```java
public final void measue(int widthMeasureSpec, int heightMeasureSpec);

protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)；

protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec)

protected void measureChild(View child, int parentWidthMeasureSpec, int parentHeightMeasureSpec)

protected void measureChildWithMargins(View child,int parentWidthMeasureSpec, int widthUsed, int parentHeightMeasureSpec, int heightUsed)
```



   **1、measure**

 接着我们来看View类中measure和onMeasure函数的源码：

```java
public final void measure(int widthMeasureSpec, int heightMeasureSpec) {  
        if ((mPrivateFlags & FORCE_LAYOUT) == FORCE_LAYOUT ||  
                widthMeasureSpec != mOldWidthMeasureSpec ||  
                heightMeasureSpec != mOldHeightMeasureSpec) {  
  
            // first clears the measured dimension flag  
            mPrivateFlags &= ~MEASURED_DIMENSION_SET;  
  
            if (ViewDebug.TRACE_HIERARCHY) {  
                ViewDebug.trace(this, ViewDebug.HierarchyTraceType.ON_MEASURE);  
            }  
  
            // measure ourselves, this should set the measured dimension flag back  
            onMeasure(widthMeasureSpec, heightMeasureSpec);  
  
            // flag not set, setMeasuredDimension() was not invoked, we raise  
            // an exception to warn the developer  
            if ((mPrivateFlags & MEASURED_DIMENSION_SET) != MEASURED_DIMENSION_SET) {  
                throw new IllegalStateException("onMeasure() did not set the"  
                        + " measured dimension by calling"  
                        + " setMeasuredDimension()");  
            }  
  
            mPrivateFlags |= LAYOUT_REQUIRED;  
        }  
  
        mOldWidthMeasureSpec = widthMeasureSpec;  
        mOldHeightMeasureSpec = heightMeasureSpec;  
    }
```

  

​    由于函数原型中有final字段，那么measure根本没打算被子类继承，也就是说measure的过程是固定的，而measure中调用了onMeasure函数，因此真正有变数的是onMeasure函数，onMeasure的默认实现很简单，源码如下：

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {  
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),  
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));  
}  
```

​    onMeasure默认的实现仅仅调用了setMeasuredDimension，setMeasuredDimension函数是一个很关键的函数，它对View的成员变量mMeasuredWidth和mMeasuredHeight变量赋值，而measure的主要目的就是对View树中的每个View的mMeasuredWidth和mMeasuredHeight进行赋值，一旦这两个变量被赋值，则意味着该View的测量工作结束。

```java
protected final void setMeasuredDimension(int measuredWidth, int measuredHeight) {  
        mMeasuredWidth = measuredWidth;  
        mMeasuredHeight = measuredHeight;  
  
        mPrivateFlags |= MEASURED_DIMENSION_SET;  
}  
```

   对于非ViewGroup的View而言，通过调用上面默认的measure——>onMeasure，即可完成View的测量，当然你也可以重载onMeasure，并调用setMeasuredDimension来设置任意大小的布局，但一般不这么做。

   对于ViewGroup的子类而言，往往会重载onMeasure函数负责其children的measure工作，重载时不要忘记调用setMeasuredDimension来设置自身的mMeasuredWidth和mMeasuredHeight。如果我们在layout的时候不需要依赖子视图的大小，那么不重载onMeasure也可以，但是必须重载onLayout来安排子视图的位置。

   再来看下measure(int widthMeasureSpec, int heightMeasureSpec)中的两个参数， 这两个参数分别是父视图提供的测量规格，当父视图调用子视图的measure函数对子视图进行测量时，会传入这两个参数，通过这两个参数以及子视图本身的LayoutParams来共同决定子视图的测量规格，在ViewGroup的measureChildWithMargins函数中体现了这个过程。

MeasureSpec参数的值为int型，分为高32位和低16为，高32位保存的是specMode，低16位表示specSize，specMode分三种：

 1、MeasureSpec.UNSPECIFIED,父视图不对子视图施加任何限制，子视图可以得到任意想要的大小；

2、MeasureSpec.EXACTLY，父视图希望子视图的大小是specSize中指定的大小；

3、MeasureSpec.AT_MOST，子视图的大小最多是specSize中的大小。

   以上施加的限制只是父视图“希望”子视图的大小按MeasureSpec中描述的那样，但是子视图的具体大小取决于多方面的。

   ViewGroup中定义了measureChildren, measureChild,  measureChildWithMargins来对子视图进行测量，measureChildren内部只是循环调用measureChild，measureChild和measureChildWithMargins的区别就是是否把margin和padding也作为子视图的大小，我们主要分析measureChildWithMargins的执行过程：

```java
protected void measureChildWithMargins(View child,  
        int parentWidthMeasureSpec, int widthUsed,  
        int parentHeightMeasureSpec, int heightUsed) {  
    final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();  
  
    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,  
            mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin  
                    + widthUsed, lp.width);  
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,  
             mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin  
                     + heightUsed, lp.height);  
   
     child.measure(childWidthMeasureSpec, childHeightMeasureSpec);  
}  
```

总的来看该函数就是对父视图提供的measureSpec参数进行了调整（结合自身的LayoutParams参数），然后再来调用child.measure()函数，具体通过函数getChildMeasureSpec来进行参数调整，过程如下：

```java
public static int getChildMeasureSpec(int spec, int padding, int childDimension) {  
    int specMode = MeasureSpec.getMode(spec);  
    int specSize = MeasureSpec.getSize(spec);  

    int size = Math.max(0, specSize - padding);  

    int resultSize = 0;  
    int resultMode = 0;  

     switch (specMode) {  
     // Parent has imposed an exact size on us  
     case MeasureSpec.EXACTLY:  
         if (childDimension >= 0) {  
             resultSize = childDimension;  
             resultMode = MeasureSpec.EXACTLY;  
         } else if (childDimension == LayoutParams.MATCH_PARENT) {  
             // Child wants to be our size. So be it.  
             resultSize = size;  
             resultMode = MeasureSpec.EXACTLY;  
         } else if (childDimension == LayoutParams.WRAP_CONTENT) {  
             // Child wants to determine its own size. It can't be  
             // bigger than us.  
             resultSize = size;  
             resultMode = MeasureSpec.AT_MOST;  
             }  
         break;  

     // Parent has imposed a maximum size on us  
     case MeasureSpec.AT_MOST:  
         if (childDimension >= 0) {  
            // Child wants a specific size... so be it  
            resultSize = childDimension;  
            resultMode = MeasureSpec.EXACTLY;  
        } else if (childDimension == LayoutParams.MATCH_PARENT) {  
            // Child wants to be our size, but our size is not fixed.  
            // Constrain child to not be bigger than us.  
            resultSize = size;  
            resultMode = MeasureSpec.AT_MOST;  
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {  
            // Child wants to determine its own size. It can't be  
            // bigger than us.  
            resultSize = size;  
            resultMode = MeasureSpec.AT_MOST;  
        }  
        break;  

    // Parent asked to see how big we want to be  
    case MeasureSpec.UNSPECIFIED:  
        if (childDimension >= 0) {  
            // Child wants a specific size... let him have it  
            resultSize = childDimension;  
            resultMode = MeasureSpec.EXACTLY;  
        } else if (childDimension == LayoutParams.MATCH_PARENT) {  
                // Child wants to be our size... find out how big it should  
                // be  
                resultSize = 0;  
            resultMode = MeasureSpec.UNSPECIFIED;  
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {  
            // Child wants to determine its own size.... find out how  
            // big it should be  
            resultSize = 0;  
            resultMode = MeasureSpec.UNSPECIFIED;  
        }  
        break;  
    }  
    return MeasureSpec.makeMeasureSpec(resultSize, resultMode);  
}  

```

 

   getChildMeasureSpec的总体思路就是通过其父视图提供的MeasureSpec参数得到specMode和specSize，并根据计算出来的specMode以及子视图的childDimension（layout_width和layout_height中定义的）来计算自身的measureSpec，如果其本身包含子视图，则计算出来的measureSpec将作为调用其子视图measure函数的参数，同时也作为自身调用setMeasuredDimension的参数，如果其不包含子视图则默认情况下最终会调用onMeasure的默认实现，并最终调用到setMeasuredDimension，而该函数的参数正是这里计算出来的。

   **2、layout**

   正如layout的中文意思“布局”中表达的一样，layout的过程就是确定View在屏幕上显示的具体位置，在代码中就是设置其成员变量mLeft，mTop，mRight，mBottom的值，这几个值构成的矩形区域就是该View显示的位置，不过这里的具体位置都是相对与父视图的位置。与onMeasure过程类似，ViewGroup在onLayout函数中通过调用其children的layout函数来设置子视图相对与父视图中的位置，具体位置由函数layout的参数决定，当我们继承ViewGroup时必须重载onLayout函数（ViewGroup中onLayout是abstract修饰），然而onMeasure并不要求必须重载，因为相对与layout来说，measure过程并不是必须的，具体后面会提到。首先我们来看下View.[Java](http://lib.csdn.net/base/java)中函数layout和onLayout的源码：

```java
public void layout(int l, int t, int r, int b) {  
       int oldL = mLeft;  
       int oldT = mTop;  
       int oldB = mBottom;  
       int oldR = mRight;  
       boolean changed = setFrame(l, t, r, b);  
       if (changed || (mPrivateFlags & LAYOUT_REQUIRED) == LAYOUT_REQUIRED) {  
           if (ViewDebug.TRACE_HIERARCHY) {  
               ViewDebug.trace(this, ViewDebug.HierarchyTraceType.ON_LAYOUT);  
           }  

           onLayout(changed, l, t, r, b);  
           mPrivateFlags &= ~LAYOUT_REQUIRED;  

           ListenerInfo li = mListenerInfo;  
           if (li != null && li.mOnLayoutChangeListeners != null) {  
               ArrayList<OnLayoutChangeListener> listenersCopy =  
                       (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();  
               int numListeners = listenersCopy.size();  
               for (int i = 0; i < numListeners; ++i) {  
                   listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);  
               }  
           }  
       }  
       mPrivateFlags &= ~FORCE_LAYOUT;  
   }  
```
函数layout的主体过程还是很容易理解的，首先通过调用setFrame函数来对4个成员变量（mLeft，mTop，mRight，mBottom）赋值，然后回调onLayout函数，最后回调所有注册过的listener的onLayoutChange函数。对于View来说，onLayout只是一个空实现，一般情况下我们也不需要重载该函数：

```java
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {  
}  
```

接着我们来看下ViewGroup.java中layout的源码：

```java
public final void layout(int l, int t, int r, int b) {  
       if (mTransition == null || !mTransition.isChangingLayout()) {  
           super.layout(l, t, r, b);  
       } else {  
           // record the fact that we noop'd it; request layout when transition finishes  
           mLayoutSuppressed = true;  
       }  
   }  
```

  super.layout(l, t, r, b)调用的即是View.java中的layout函数，相比之下ViewGroup增加了LayoutTransition的处理，LayoutTransition是用于处理ViewGroup增加和删除子视图的动画效果，也就是说如果当前ViewGroup未添加LayoutTransition动画，或者LayoutTransition动画此刻并未运行，那么调用super.layout(l, t, r, b)，继而调用到ViewGroup中的onLayout，否则将mLayoutSuppressed设置为true，等待动画完成时再调用requestLayout()。上面super.layout(l, t, r, b)会调用到ViewGroup.java中onLayout，其源码实现如下：

```java
@Override  
 protected abstract void onLayout(boolean changed,int l, int t, int r, int b);  
```

   和前面View.java中的onLayout实现相比，唯一的差别就是ViewGroup中多了关键字abstract的修饰，也就是说ViewGroup类只能用来被继承，无法实例化，并且其子类必须重载onLayout函数，而重载onLayout的目的就是安排其children在父视图的具体位置。重载onLayout通常做法就是起一个for循环调用每一个子视图的layout(l, t, r, b)函数，传入不同的参数l, t, r, b来确定每个子视图在父视图中的显示位置。 那layout(l, t, r, b)中的4个参数l, t, r, b如何来确定呢？联想到之前的measure过程，measure过程的最终结果就是确定了每个视图的mMeasuredWidth和mMeasuredHeight，这两个参数可以简单理解为视图期望在屏幕上显示的宽和高，而这两个参数为layout过程提供了一个很重要的依据（但不是必须的），为了说明这个过程，我们来看下LinearLayout的layout过程：

```java
void layoutVertical() {  
    ……  
   for (int i = 0; i < count; i++) {  
        final View child = getVirtualChildAt(i);  
        if (child == null) {  
            childTop += measureNullChild(i);  
        } else if (child.getVisibility() != GONE) {  
            final int childWidth = child.getMeasuredWidth();  
            final int childHeight = child.getMeasuredHeight();  
            ……  
            setChildFrame(child, childLeft, childTop + getLocationOffset(child),  
                    childWidth, childHeight);  
            childTop += childHeight + lp.bottomMargin + getNextLocationOffset(child);  
 
            i += getChildrenSkipCount(child, i);  
        }  
    }  
   
private void setChildFrame(View child, int left, int top, int width, int height) {          
        child.layout(left, top, left + width, top + height);  
    }  
```

从setChildFrame可以看到LinearLayout中的子视图的右边界等于left + width，下边界等于top+height，也就是说在LinearLayout中其子视图显示的宽和高由measure过程来决定的，因此measure过程的意义就是为layout过程提供视图显示范围的参考值。

layout过程必须要依靠measure计算出来的mMeasuredWidth和mMeasuredHeight来决定视图的显示大小吗？事实并非如此，layout过程中的4个参数l, t, r, b完全可以由视图设计者任意指定，而最终视图的布局位置和大小完全由这4个参数决定，measure过程得到的mMeasuredWidth和mMeasuredHeight提供了视图大小的值，但我们完全可以不使用这两个值，可见measure过程并不是必须的。

说到这里就不得不提getWidth()、getHeight()和getMeasuredWidth()、getMeasuredHeight()这两对函数之间的区别，getMeasuredWidth()、getMeasuredHeight()返回的是measure过程得到的mMeasuredWidth和mMeasuredHeight的值，而getWidth()和getHeight()返回的是mRight  - mLeft和mBottom - mTop的值，看View.java中的源码便一清二楚了：

```java
public final int getMeasuredWidth() {  
        return mMeasuredWidth & MEASURED_SIZE_MASK;  
    }  
public final int getWidth() {  
        return mRight - mLeft;  
    }  
```

   这也解释了为什么有些情况下getWidth()和getMeasuredWidth()以及getHeight()和getMeasuredHeight()会得到不同的值。整个layout过程比较容易理解，一般情况下layout过程会参考measure过程中计算得到的mMeasuredWidth和mMeasuredHeight来安排子视图在父视图中显示的位置，但这不是必须的，measure过程得到的结果可能完全没有实际用处，特别是对于一些自定义的ViewGroup，其子视图的个数、位置和大小都是固定的，这时候我们可以忽略整个measure过程，只在layout函数中传入的4个参数来安排每个子视图的具体位置。

  ** 3、draw**

和measure和layout一样，draw过程也是在ViewRoot的performTraversals()的内部发起的，其调用顺序在measure()和layout()之后，同样的，performTraversals()发起的draw过程最终会调用到mView的draw()函数，这里的mView对于Actiity来说就是PhoneWindow.DecorView。

 

     首先来看下与draw过程相关的函数

ViewRootImpl.draw()，仅在ViewRootImpl.performTraversals()的内部调用

DecorView.draw(), 上一步中的ViewRootImpl.draw()会调用到该函数，DecorView.draw()继承自Framelayout，DecorView和FrameLayout，以及FrameLayout的父类ViewGroup都未重载draw(),而ViewGroup的父类是View类，因此DecorView.draw()调用的是View.draw()的默认实现

View.onDraw()，绘制View本身，自定义View往往会重载该函数来绘制View本身的内容

View.dispatchDraw(), View中的dispatchDraw默认是空实现，ViewGroup重载了该函数，内部会循环调用View.drawChild()来发起对子视图的绘制，应用程序不应该重载ViewGroup，因为该函数的默认实现代表了View的绘制流程

ViewGroup.drawChild()，该函数只在ViewGroup中实现，原因就是只有ViewGroup才需要绘制child，drawChild内部又会调用View.draw()函数来完成子视图的绘制（有可能直接调用dispatchDraw）

先从源码来展现draw的整体过程，当然源码中只展现关键部分，首先来看performTraversals()，因为draw过程最先是从这里发起的：

```java
 void performTraversals() {  
    final View host = mView;  
    ...  
    host.measure(childWidthMeasureSpec, childHeightMeasureSpec);  
    ...  
    host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());  
    ...  
    draw(fullRedrawNeeded);  
}  
```
   注意到measure和layout过程直接调用的是mView的measure和layout函数，而draw调用的是ViewRootImpl的内部draw(boolean fullRedrawNeeded)函数，再由draw(boolean fullRedrawNeeded)函数来调用mView.draw()函数，draw(boolean fullRedrawNeeded)包含draw过程的一些前期处理，通过下面的代码可以看出调用关系：

```java

private void draw(boolean fullRedrawNeeded) {  
        Surface surface = mSurface;  
        if (surface == null || !surface.isValid()) {  
            return;  
        }  
    ...  
        try {  
         canvas.translate(0, -yoff);  
         if (mTranslator != null) {  
          mTranslator.translateCanvas(canvas);  
         }  
         canvas.setScreenDensity(scalingRequired  
                   ? DisplayMetrics.DENSITY_DEVICE : 0);  
         mAttachInfo.mSetIgnoreDirtyState = false;  
         mView.draw(canvas);  
       } finally {  
        if (!mAttachInfo.mSetIgnoreDirtyState) {  
        // Only clear the flag if it was not set during the mView.draw() call  
        mAttachInfo.mIgnoreDirtyState = false;  
         }  
      }  
    ...  
}  
```

下面将转到mView.draw(),之前提到mView.draw()调用的就是View.java的默认实现，View类中的draw函数体现了View绘制的核心流程，因此我们下面重点来看下View.java中draw的调用流程：


```java

public void draw(Canvas canvas) {  
   ...  
       /*  
        * Draw traversal performs several drawing steps which must be executed  
        * in the appropriate order:  
        *  
        *      1. Draw the background  
        *      2. If necessary, save the canvas' layers to prepare for fading  
        *      3. Draw view's content  
        *      4. Draw children  
        *      5. If necessary, draw the fading edges and restore layers  
        *      6. Draw decorations (scrollbars for instance)  
        */  

       // Step 1, draw the background, if needed  
   ...  
       background.draw(canvas);  
   ...  
       // skip step 2 & 5 if possible (common case)  
   ...  
       // Step 2, save the canvas' layers  
   ...  
       if (solidColor == 0) {  
           final int flags = Canvas.HAS_ALPHA_LAYER_SAVE_FLAG;  
  
            if (drawTop) {  
                canvas.saveLayer(left, top, right, top + length, null, flags);  
            }  
    ...  
        // Step 3, draw the content  
        if (!dirtyOpaque) onDraw(canvas);  
 
        // Step 4, draw the children  
        dispatchDraw(canvas);  
 
        // Step 5, draw the fade effect and restore layers  
 
        if (drawTop) {  
            matrix.setScale(1, fadeHeight * topFadeStrength);  
            matrix.postTranslate(left, top);  
            fade.setLocalMatrix(matrix);  
            canvas.drawRect(left, top, right, top + length, p);  
        }  
    ...  
        // Step 6, draw decorations (scrollbars)  
        onDrawScrollBars(canvas);  
    }  
```

   通过阅读上面的代码可以知道整个绘制过程包括View的背景绘制，View本身内容的绘制，子视图的绘制（如果包含子视图），渐变框的绘制以及滚动条的绘制。重点要关注的是View本身内容的绘制和子视图的绘制，即onDraw()和dispatchDraw()函数。对于View.java和ViewGroup.java，onDraw()默认都是空实现，因为具体View本身长什么样子是由View的设计者来决定的，默认不显示任何东西。

​    View.java中dispatchDraw()默认为空实现，因为其不包含子视图，而ViewGroup重载了dispatchDraw()来对其子视图进行绘制，通常应用程序不应该对dispatchDraw()进行重载，其默认实现体现了View系统绘制的流程。那么，接下来我们继续分析下ViewGroup中dispatchDraw()的具体流程：

```java
@Override  
    protected void dispatchDraw(Canvas canvas) {  
       ...  
 
        if ((flags & FLAG_USE_CHILD_DRAWING_ORDER) == 0) {  
            for (int i = 0; i < count; i++) {  
                final View child = children[i];  
                if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE || child.getAnimation() != null) {  
                    more |= drawChild(canvas, child, drawingTime);  
                }  
            }  
        } else {  
            for (int i = 0; i < count; i++) {  
                final View child = children[getChildDrawingOrder(count, i)];  
                if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE || child.getAnimation() != null) {  
                    more |= drawChild(canvas, child, drawingTime);  
                }  
            }  
        }  
      ......  
    }  
```

dispatchDraw()的核心代码就是通过for循环调用drawChild()对ViewGroup的每个子视图进行绘制，上述代码中如果FLAG_USE_CHILD_DRAWING_ORDER为true，则子视图的绘制顺序通过getChildDrawingOrder来决定，默认的绘制顺序即是子视图加入ViewGroup的顺序，而我们可以重载getChildDrawingOrder函数来更改默认的绘制顺序，这会影响到子视图之间的重叠关系。

drawChild()的核心过程就是为子视图分配合适的cavas剪切区，剪切区的大小正是由layout过程决定的，而剪切区的位置取决于滚动值以及子视图当前的动画。设置完剪切区后就会调用子视图的draw()函数进行具体的绘制，如果子视图的包含SKIP_DRAW标识，那么仅调用dispatchDraw()，即跳过子视图本身的绘制，但要绘制视图可能包含的字视图。完成了dispatchDraw()过程后，View系统会调用onDrawScrollBars()来绘制滚动条。



参考博客：

[http://blog.csdn.net/zjmdp/article/details/7713963](http://blog.csdn.net/zjmdp/article/details/7713963)

[http://blog.csdn.net/qinjuning/article/details/7110211](http://blog.csdn.net/qinjuning/article/details/7110211)

[http://www.cnblogs.com/franksunny/archive/2012/04/20/2459738.html](http://www.cnblogs.com/franksunny/archive/2012/04/20/2459738.html)