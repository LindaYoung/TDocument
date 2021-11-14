一、事件传递机制



### 1. 事件传递顺序

activity -> window -> decor view -> contentview

![image-20210107183451688](/Users/young1/Library/Application Support/typora-user-images/image-20210107183451688.png)





#### 修改子view绘制顺序

[修改子view绘制顺序](https://mp.weixin.qq.com/s/nEvBqMPrdmirADo-Y6ihYg)

总结：

- ViewGroup可以通过调用setChildrenDrawingOrderEnable(true)方法，以及重写getChildDrawingOrder方法修改子view绘制顺序。
- Recyclerview中将两者进行了封装，只需要调用setChildDrawingOrderCallback方法即可完成子view绘制顺序的需求。
- 事件分发的过程中，遍历子view的顺序和绘制子view的顺序获取机制是相同的。（只是两者的顺序获取机制是相同的，都是通过getChildDrawingOrder方法获取，但是两者顺序并不是完全相同的。因为事件分发中遍历子view是倒序的，也就是从最后一个view开始遍历，而绘制子view的顺序是正序，也就是从第一个view开始遍历）
- 所以，外面修改子view绘制顺序的同时，其实也修改了事件分发的子view的遍历顺序。



### 2.事件传递规则

- dispatchTouchEvent(MotionEvent ev) - 判断是否分发事件，return true向上分发，return false不向上分发。返回false后默认执行ontouchEvent事件
- onInterceptTouchEvent(MotionEvent ev) - 判断是否拦截了某个事件，return true拦截，不向下分发，默认执行onTouchEvent事件。return false不拦截，向下分发
- onTouchEvent(MotionEvent ev) - 判断是否消耗了某个事件，return true表示消耗了事件，return false表示没有消耗事件，会继续向父布局的onTouchEvent事件传递。

只有viewgroup才有onInterceptTouchEvent，view和activity都没有此方法。

![image-20210107191111654](/Users/young1/Library/Application Support/typora-user-images/image-20210107191111654.png)

把activity看成最底层，事件dispatch是从下到向上的一个过程，而onTouch事件是从上到下的一个过程。

activity的事件传递：

![image-20210108111728196](/Users/young1/Library/Application Support/typora-user-images/image-20210108111728196.png)



viewGroup的事件传递

![image-20210108113013098](/Users/young1/Library/Application Support/typora-user-images/image-20210108113013098.png)

[工匠若水仔细看](https://blog.csdn.net/yanbober/article/details/45887547)

### 3.源码分析

app层事件传递是从activity开始的，首先看调用activity的dispatchTouchEvent。

#### activity对事件的传递

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
      // activity的public空方法，如果在事件开始传递前需要额外处理一些操作，可以在onUserInteraction()中进行处理。应用场景：屏保功能
        onUserInteraction();
    }
    if (getWindow().superDispatchTouchEvent(ev)) { 
      //getWindow().superDispatchTouchEvent返回的就是DecorView事件处理，相当于最顶层的ViewGroup，然后就开始往下层的videw传递，如果事件在view的传递中被处理，则返回true。
      //activity 的dispatchToucheEvent返回true，则方法结束，即：该点击事件停止往下传递&事件传递过程结束，否则继续往下调用activity.onTouch
        return true;
    }
  //如果事件在view的传递中未被处理，则调用activity自己的onTouchEvent方法。
    return onTouchEvent(ev); 
}
```



#### view对事件的传递

小记：我们会通过decorview获取activity的布局view：(ViewGroup)getWindow().getDecorView().findViewById(android.R.id.content).getChild(0);





- 在一个完整的事件序列中，当某个子view拦截ACTION_DOWN后，后续所有事件都会交给它处理，而父view也不会再去调用onInterceptTouchEvent方法
- 当新来到一个事件序列时（也就是以ACTION_DOWN开头），会重置mFirstTouchTarget以及标记为FLAG_DISALLOW_INTERCEPT等



 [参考链接1](https://blog.csdn.net/qq475703980/article/details/92385368)

[参考链接2](https://www.jianshu.com/p/38015afcdb58)



# 二、基础知识

#### 五种布局

FrameLayout、LinearLayout、RelativeLayout、TableLayout、AbsoluteLayout 全部继承于ViewGroup。

#### Activity生命周期

![image-20210220113626062](/Users/young1/Library/Application Support/typora-user-images/image-20210220113626062.png)

- 启动Activity：onCreate -> onStart -> onResume ，Activity进入运行状态
- Activity退居后台：onPause -> onStop，进入停滞状态
- Activity返回前台：onRestart -> onStart -> onResume ，再次回到运行状态
- Activity退居后台，且系统内存不足，系统会杀死这个后台状态的Activity（此时这个Activity引用仍然在任务栈中，但引用指向的对象已经为null），若再次回到这个Activity，则会走onCreate -> onStart -> onResume
- 锁屏： onPause -> onStop
- 解屏： onStart -> onResume

#### Activity 任务栈

使用android:launchMode="standard|singleInstance|singleTask|singleTop"来控制Activity任务栈。

任务栈是一种后进先出的结构。每个APP默认只有一个栈，以APP的包名来命名。

- standard：标准模式，每次启动Activity都会创建一个新的Activity实例，并且将其压入任务栈栈顶，而不管这个Activity是否已经存在。Activity的启动三回调（onCreate -> onStart ->onResume ）都会执行。
- singleTop：栈顶复用模式，创建Activity时先去确认，要创建的Activity是否在栈顶，如果在任务栈栈顶，那么此Activity不会被重新创建而是被复用，它的启动三回调就不会执行，同时Activity的onNewInstance()方法会被回调；如果新的Activity不在栈顶则会创建一个新的Activity实例压入栈顶。<服务详情页>
- singleTask：栈内复用模式，创建Activity时系统会先确定它所需的任务栈是否已经创建，如果没有则创建这个任务栈并把Activity实例压入栈顶；如果已经被创建，那么这个Activity就会被调到栈顶，并且会回调onNewInstance()，而且还会清理在当前Activity上面的所有Activity(clear top)。<首页>
- singleInstance：加强版的singleTask模式，这种模式的Activity只能单独位于一个任务栈内，由于栈内复用的特性，后续请求均不会创建新的Activity，除非这个独特的任务栈被系统销毁了

Activity的堆栈管理以ActivityRecord为单位，所有的ActivityRecord都放在一个List里面，可以认为一个ActivityRecord就是一个Activity栈。

#### Activity缓存方法

可以用Activity中的onSaveInstanceState回调方法保存临时数据和状态，这个方法一定会在活动被回收之前调用。方法中有一个Bundle参数可以用来保存数据，之后会在onCreate中恢复，onCreate中也有一个Bundle的参数 

##### 一、onSaveInstanceState(Bundle outState)

当某个activity变得“容易”被系统销毁时（表示还没有被销毁，仅仅是有一种被销毁的可能），该activity的onSaveInstanceState就会被执行，除非该acitivity被用户主动销毁的（back键）。

该方法被调用的情况：

1. 当用户按下HOME键时：系统不知道你按下HOME后要运行多少其它的程序，自然也不知道activity A是否会被销毁，故系统会调用onSaveInstanceState，让用户有机会保存某些非永久性的数据。
2. 长按HOME键，选择运行其它的程序时
3. 按下电源按键（关闭屏幕显示）时
4. 从activity A中启动一个新的activity时
5. 屏幕方向切换时。如果不置顶configchange属性在屏幕切换之前，系统会销毁activity A，在屏幕切换之后系统又会自动地创建activity A，所以onSaveInstanceState一定会被执行

总而言之，onSaveInstanceState的调用遵循一个重要原则，即当系统“未经你许可”时销毁了你的activity，则onSaveInstanceState会被系统调用，这是系统的责任，因为它必须要提供一个机会让你保存你的数据。需要注意以下几点：

a. 布局中的每一个View默认实现了onSaveInstanceState方法，这样的话，这个UI的任何改变都会自动地存储和在activity重新创建的时候自动地恢复。但这种情况只有在你为这个UI提供了唯一的ID之后才起作用，如果没有提供ID，app将不会存储它的状态。

b. 由于默认的onSaveInstanceState方法的实现帮助UI存储它的状态，所以如果你需要覆盖这个方法存储额外的状态信息，你应该在执行任何代码之前都调用父类的onSaveInstanceState方法（super.onSaveInstanceState()）。

c. 由于onSaveInstanceState方法调用的不确定性，你应该只使用这个方法去记录activity的瞬间状态（UI状态）。不应该用这个方法去存储持久化数据。当用户离开这个activity的时候应该在onPauce方法中存储持久化数据。

d. onSaveInstanceState如果被调用，**这个方法会在onStop之前被触发，但系统并不保证在onPause之前或者之后触发。**

##### 二、onRestoreInstanceState(Bundle outState)

注意：onRestoreInstanceState和onSaveInstanceState方法不一定是成对被调用的。

onRestoreInstanceState被调用的前提是activity A确实被系统销毁了。onRestoreInstanceState的bundle参数也会传递到onCreate方法中，也可以选择在onCreate方法中做数据还原。还有onRestoreInstanceState在onStart之后执行。

#### Activity横竖屏切换生命周期

```java
android:configChanges="mcc|mnc|locale|touchscreen|keyboard|keyboardHidden|navigation|screenLayout|fontScale|uiMode|orientation|screenSize|smallestScreenSize|layoutDirection"
```

- 如果configchanges未设置，且指定了screenOrientation：

onCreate --> onStart --> onResume --> onPause --> onStop -->onSaveInstanceState --> onCreate --> onStart --> onRestoreInstanceState --> onResume

注意：这种情况下onConfigChanged方法不调用

- 如果配置了configchanges可以使屏幕切换时不走生命周期，但会回调onConfigChanged方法。



#### Fragment

##### Fragment的生命周期

![image-20210220144022318](/Users/young1/Library/Application Support/typora-user-images/image-20210220144022318.png)

和Activity的生命周期相比的区别多了onAttach，onDetach 以及 onCreateView, onDestroyView和onActivityCreated。

##### 为什么在Service中创建子线程而不是在Activity中

因为Activity很难对Thread进行控制，当Activity被销毁之后，就没有任何其它办法可以重新获取到之前创建的子线程实例。在ActivityA中创建的子线程 ActivityB无法对其进行操作。Service就不同了，所有的Activity都可以与Service进行关联，然后可以很方便的操作其中的方法。即使Activity被销毁了，之后只要重新与Service建立关联，就又能够获取到原有的Service中的Binder的实例。因此，使用Service来处理后台任务，Activity就可以放心地finish，完全不需要担心无法对后台任务进行控制的情况。

##### Intent的使用方法，可以传递哪些数据类型

可以传递基本类型的数据和基本类型的数组数据，以及String/CharSequence类型的数据和String/CharSequence类型的数组数据，和Parcelable（包裹化，邮化）和Serializable（ 序列化）类型的数据以及它们的数组/列表数据。

##### Parcelable 和 Serializable的区别

Serializable是java的序列化技术，使用简单，只需实现这个接口就行，不需要手动去处理序列化和反序列化的过程，常常用于网络请求数据处理，Activity之间传递值的使用。Serializable无法序列化静态变量。当一个父类实现了序列化，子类自动实现序列化，不需要再显示实现Serializable接口。

Parcelabe是android特有的序列化API，它的出现是为了解决Serializable在序列化的过程中消耗资源严重的问题，但本身使用需要手动处理序列化和反序列化过程，会与具体的代码绑定，使用较为繁琐，一般只获取内存数据时使用。

##### Service的两种启动方式有什么区别

1. 在Context中通过public boolean bindService(Intent service, ServiceConnection conn, int flags)方法来进行Service和Context的关联并启动，并且Service的生命周期依附于Context。
2. 通过public CommponentName startService(Intent service)方法来启动一个Service，此时Service的生命周期与启动它的Context无关。



![image-20210220145254959](/Users/young1/Library/Application Support/typora-user-images/image-20210220145254959.png)



Service是四大组件之一，需要在xml里注册你的Service

```java
<service
  android:name=".packageName.yourServicename"
  android:enabled="true"/>
```

[Android Service的两种启动模式](https://www.jianshu.com/p/4c798c91a613)

##### Service保活

1. Service 设置成START_STICKY

- kill后会被启动（等待5s左右），重传Intent，保持与重启前一样

2. 提升Service进程优先级

- Android中进程是托管的，当系统进程空间紧张的时候，会依照优先级自动进行进程的回收
- 当service运行在低内存的环境时，将会kill掉一些存在的进程。因此进程的优先级将会很重要，可以使用startForeground()将service放到前台状态。
- ⚠️如果在极度低内存的压力下，该service还会被kill掉，并且不一定会restart（）

3. onDestroy方法里重启service

- service+broadcast方式，就是当service走onDestroy的时候，发送一个自定义的广播，当收到广播的时候，重新启动service
- 直接在onDestroy里startService
- ⚠️当使用类似口口管家等第三方应用或事在setting里应用 强制停止时，APP进程可能就直接被干掉了，onDestroy方法都进不来，所以还是无法保证

4. 监听系统广播判断service状态

- 通过系统广播（比如手机重启、界面唤醒、应用状态改变）等等监听并捕获到，然后判断我们的service是否还存活（需要加权限）
- ⚠️这只能算是一种措施，如果监听多了会导致service比较混乱

5. JNI层用C代码fork一个进程出来

   这样产生的进程，会被系统认为是两个不同的进程，但在Android5.0之后可能不行

6. 让一个像素在前台（手机QQ）

##### 广播的两种注册状态的区别

- 静态注册：在AndroidManifest.xml文件中进行注册，当App退出后，Receiver仍然可以接收到广播并且进行相应的处理
- 动态注册：代码中动态注册，当App退出后，也就没办法再接受广播了

##### ContentProvider的使用

- 作用：将数据共享给其他应用

[关于ContentProvider的使用](http://blog.csdn.net/juetion/article/details/17481039)

##### 动画有哪两类，各有什么特点？三种动画的区别

- 补间动画：在Android3.0之前（API 11）前的，补间动画的特点是只能作用于view上。动画效果只是视觉上的效果，其view组件还是在原来的位置。

- 帧动画；像电影片段一样，由一张一张的图片组成的一组动画效果，效果比较多样化，实现种类多，缺点是资源消耗过大，繁琐。

- 属性动画：属性动画是android3.0之后出来的，其目的就是改善，解决补间动画的不足，所以属性动画的使用的地方就比较广泛，不单单是简单的view上，而是java对象，并且组件的位置也会根据动画效果而改变。实现效果多样化。

  属性动画通过不断对值发生改变，并不断把该值赋值给对象的属性，从而实现该对象在该属性上的动画效果。用到两个方法类：ValueAnimator和ObjectAnimator；两个辅助使用类：差值器和估值器。

##### Android的数据存储形式

- SQLite： 轻量级的数据库，支持基本的SQL语法，是常被采用的一种数据存储方式。Android为此数据库提供了一个名为SQLiteDatabase的类，封装了一些操作数据库的api
- SharedPreference：其本质就是一个xml文件，常用于存储较简单的参数设置
- File：常用于存储大数量的数据，但缺点是更新数据将是一件困难的事情
- ContentProvider：Android系统中能实现所有应用程序共享的一种数据存储方式，由于数据通常在各应用间的是互相私密的，所以此存储方式较少使用，但是其必不可少的一种存储方式。例如：音频、视频、图片和通讯录等。

##### 如何判断应用被强杀

在Application中定义个static常量，赋值为-1，在欢迎界面改为0，如果被强杀，application重新初始化，在父类的Activity判断该常量的值。

##### Asset目录和res目录的区别

res目录下面有很多文件，drawable，mipmap，raw等，除了raw文件不会被压缩外，其余文件都会被压缩。同时res目录下的文件可以通过R文件访问。Asset也是用来存储资源的，但Asset文件内容只能通过路径或者AssetManager读取。

##### Android怎么加速启动Activity

启动应用：Application的构造方法，onCreate方法中不要进行耗时操作，数据预读取放在异步中操作。

启动普通activity：A启动B时，不要在A的onPause中执行耗时操作，因为B的onResume方法必须等待A的onPause执行完成之后才能运行。

##### Apk打包流程



##### Apk安装流程



##### APP启动过程





##### WebView cookie问题

###### cookie同步到WebView，webview的cookie机制

- WebView是基于webkit内核的UI控件，相当于一个浏览器客户端。它会在本地维护每次会话的cookie（保存在data/data/package_name/app_WebView/Cookies.db)

- WebView加载URL的时候，WebView会从本地读取该URL对应的cookie，并携带该cookies与服务器进行通信

- WebView通过android.webkit.CookieManager类来维护cookie。

  [参考链接](https://www.jianshu.com/p/24827940b21a/)

##### java对引用对分类

​										回收机制 --> 用途 --> 生存时间

- strongReference： 从来不回收 --> 是对象的一般状态 --> JVM停止运行时终止
- softReference：在内存不足时回收 --> 联合ReferenceQueue构造有效期短/占用内存大/生命周期长的对象的二级高速缓冲器-->内存不足时终止
- weakReference: 在垃圾回收时回收-->联合ReferenceQueue构造有效期短/占用内存大/生命周期长的对象的一级高速缓冲器-->gc运行后终止
- phatomReference（虚引用）：在垃圾回收时回收-->联合ReferenceQueue来跟踪对象被垃圾回收器回收的活动--> gc运行后终止

在Android应用的开发中，为了防止内存溢出，在处理一些占用内存大而且生命周期较长的对象的时候，可以尽量应用软引用和弱引用。如果经常使用就用软引用，如果该对象不被使用的可能性更大些就用弱引用。

##### 软键盘

软键盘是一个dialog，系统api提供了 显示/隐藏的方法，但没有显示、隐藏的监听，这时候用GloableLayoutListener来监听判断。

```kotlin
class SoftKeyboardUtil {
    companion object{
        /**
         *  显示软键盘
          */
        fun showSoftInputkeyboard(view: View?) {
            if (view == null) return
            // view 一般是edittext，如果是其它的，需要额外做些工作才能弹出键盘
            if (view is EditText) {
                (view.context.getSystemService(Context.INPUT_METHOD_SERVICE) as InputMethodManager).showSoftInput(view, 0)
            } else {

            }
        }

        /**
         *  关闭软键盘
         */
        fun hideSoftInputKeyboard(view: View?) {
            if (view == null) return
            (view.context.getSystemService(Context.INPUT_METHOD_SERVICE) as InputMethodManager).hideSoftInputFromWindow(view.windowToken, 0)
        }
    }


    interface OnSoftKeyboardChangeListener {
        fun softKeyboardShow(height: Int, width: Int, left: Int)
        fun softKeyboardHide()
    }

    private var mRootViewVisibleHeight: Int = 0
    private val mVisibleRect: Rect = Rect()
    fun softKeyboardChange(rootView: View, onSoftKeyboardChangeListener: OnSoftKeyboardChangeListener) {
        rootView.viewTreeObserver.addOnGlobalLayoutListener(ViewTreeObserver.OnGlobalLayoutListener {
            rootView.getWindowVisibleDisplayFrame(mVisibleRect)
            val visibleHeight: Int = mVisibleRect.height()
            if (mRootViewVisibleHeight == 0) {
                mRootViewVisibleHeight = visibleHeight
                return@OnGlobalLayoutListener
            }

            //根视图显示高度没有变化，可以看作软键盘显示／隐藏状态没有改变
            if (mRootViewVisibleHeight == visibleHeight) {
                return@OnGlobalLayoutListener
            }

            //根视图显示高度变小超过1/3，可以看作软键盘显示了
            val height: Int = mRootViewVisibleHeight - visibleHeight
            if (height > ConstantData.DL_SCREEN_HEIGHT / 3) {
              	// 有些有虚拟导航条的，因rect变化，需要延迟显示
                HandlerHelper.getInstance().postDelayed(Runnable { onSoftKeyboardChangeListener.softKeyboardShow(visibleHeight, mVisibleRect.width(), mVisibleRect.left) }, 100)
                mRootViewVisibleHeight = visibleHeight
                return@OnGlobalLayoutListener
            }

            //根视图显示高度变大超过1/3，可以看作软键盘隐藏了
            if (visibleHeight - mRootViewVisibleHeight > ConstantData.DL_SCREEN_HEIGHT / 3) {
                onSoftKeyboardChangeListener.softKeyboardHide()
                mRootViewVisibleHeight = visibleHeight
            }
        })
    }

}
```



#### 图片中的三级缓存

1. 为什么要使用三级缓存？

   app通过网络获取图片，如果非wifi情况下会消耗大量的流量，而且每次通过网络获取图片网络状况不是很好的情况下势必加载比较慢，影响用户体验。

2. 什么是三级缓存

   - 网络加载， 不优先加载，速度慢，浪费流量
   - 本地缓存， 次优先加载，速度快
   - 内存缓存， 优先加载，速度最快
   - 弱引用？

3. 三级缓存的原理

   - 首次加载app时，肯定要通过网络交互来获取图片，之后我们可以将图片保存至本地或内存中
   - 之后运行app时，优先访问内存中的图片缓存，若内存中没有，则加载本地缓存中的图片

   [参考链接](https://www.jianshu.com/p/2cd59a79ed4a)

常用的三级缓存主要有LruCache（内存缓存）、DiskLruCache（持久化缓存）、网络。Lru表示最近最少使用，意思是当缓存到达限制时候，优先淘汰近期内最少使用的缓存。

LruCache<K,V>可以在缓存中缓存数据，内部使用最近最少使用算法，优先淘汰最近时间内最少次使用的缓存。LruCache使用LinkedHashMap来缓存，LinkedHashMap继承自HashMap，而且内部维护着一个双向队列，可以设置根据访问动作或者插入动作来调整顺序。当插入一个结点时候将该结点插入到队列的尾部或者访问某个结点时，会将该结点调整到队列尾部。这样保证当超过缓存容量的时候，直接从头部删除很久没用过的结点就可以了。



#### LayoutInflater.inflate(int resource, ViewGroup root, boolean attachToRoot)

```java
public void inflate(@LayoutRes int resource, @Nullable ViewGroup root) {
  return inflate(resource, root, root != null)
}

// 所以最终研究的是四个方法
inflate.inflate(R.layout.xx, null, true);
inflate.inflate(R.layout.xx, null, false);
inflate.inflate(R.layout.xx, parent, false);
inflate.inflate(R.layout.xx, parent, true);
```

- 第一个参数：想要添加的布局

- 第二个参数：想要添加到哪个布局上(null和非null的区别：null时第一个参数中最外层的布局大小无效，非null的时候，第一个参数中最外层的布局大小有效)

- 第三个参数：是否直接添加到第二个参数布局上（true代表layout文件填充的view会被直接添加进parent，而false则表示创建的view会以其他方式被添加进parent)

  

#### HashMap、HashTable、LinkedHashMap、TreeMap、ConcurrentHashMap、SparseArray、ArrayMap

- HashMap 是非线程安全的，HashTable是线程安全的；HashMap的键和值都允许有null值存在，HashTable则不行；因为线程安全的问题HashMap效率要比HashTable的要高。
- HashMap的结构就是一个链表数组，也就是装链表单位的数组。
- HashTable和ConcurrentHashMap都是线程安全的。HashTable是锁住整个hash表，而concurrentHashMap引入了分割，只锁当前需要用到的桶。HashTable的线程安全级别高但效率相对低。
- LinkedHashMap保存了记录的插入顺序，在用iterator遍历LinkedHashMap时，先得到的记录肯定是先插入的。
- TreeMap实现SortMap接口，能够保存的记录根据键排序，默认是按键值升序排序，也可以指定排序的比较器。TreeMap取出来的是排序后的键值对。
- SparseAarray 是Android提供的api，为了更好的节约内存。SparseArray比HashMap更节省内存，在某些条件下性能更好，主要是因为它减少了对key的自动装箱（int转为Integer类型），它内部是通过两个数组来进行数据存储的，一个存储key一个存储value，为了优化性能，它内部对数据还采取了压缩的方式来表示稀疏数组的数据，从而节约内存空间，SparseArray在存储和读取数据的时候，使用的是二分查找法。所以，数量大的情况下性能并不明显。
- ArrayMap是一个key-value映射的数据结构，它设计上更多的是考虑内存优化，内部使用两个数组进行数据存储，一个数组记录key的hash值，另一个数组记录value值，它和sparseArray一样，也会对key使用二分法进行从小到大排序。

- 链表：空间不连续，寻址困难，增删元素只需改置针，所以查询慢，增删快。
- 数组：连续空间，寻址迅速，删除或者添加元素的时候需要有大幅度的移动，所以查询快，增删慢。



## 三、经典问题

#### ANR问题

1. 出错原因

   KeyDispatchTimeout(5s) : 主要是类型按键或触摸事件在特定时间内无响应

   BroadcastTimeout(10s): BroadcastReceiver在特定时间内无法处理完成

   ServiceTimeout(20s)：小概率事件Service在特定时间内无法完成

2. 出错操作：

   在主线程里进行高耗时操作（如图像变换，磁盘读写，数据库读写，大量的创建新对象）

3. 如何避免

   UI线程尽量只做跟UI相关的工作，耗时操作放到单独的线程里处理，尽量用Handler来处理UIThread和别的Thread之间的交互

4. 如何分析ANR问题

   首先分析log；

   从trace.txt文件查看调用stack， adb  pull data/anr/traces.txt ./mytraces.txt;

   看代码，仔细查看ANR的成因；

   使用LeckCanary工具；

##### Android内存泄漏

###### Java内存分配策略

java程序运行时的内存分配策略有三种，分别是静态分配，栈式分配和堆式分配，对应三种存储策略使用的内存空间，主要分别是静态存储（也称方法区），栈区和堆区。

- 静态存储区（方法区）：主要存放静态数据、全局static数据和常量。这块内存在程序编译时就已经分配好，并且在程序整个运行期间都存在。
- 栈区：当方法被执行时，方法体内的局部变量（其中包括基础数据类型、对象和引用）都在栈上创建，并在方法执行结束时这些局部变量所持有的内存将会自动被释放。因为栈内分配运算内置于处理器的指令集中，效率很高，但是分配的内存容量有效。
- 堆区：又称动态内存分配，通常就是指在程序运行时直接new出来的内存，也就是对象的实例。这部分内存在不使用时将会由Java垃圾回收器来负责回收。

###### 栈与堆的区别

在方法体内定义的（局部变量）一些基本类型的变量和对象的引用变量都是在方法的栈内存中分配的。当在一段方法块中定义一个变量时，Java就会在栈中为该变量分配内存空间，当超过该变量的作用域后，该变量也就无效了，分配给它的内存空间也将被释放掉，该内存空间可以被重新使用。

堆内存用来存放所有由new创建的对象（包括该对象其中的所有成员变量）和数组。在堆中分配内存，将由java垃圾回收器来自动管理。在堆中产生了一个数组或者对象后，还可以在栈中定义一个特殊的变量，这个变量的取值等于数组或者对象在堆内存中的首地址，这个特殊的变量就是我们上面说的引用变量。我们可以通过这个引用变量来访问堆中国呢的对象或者数组。

```java
public class Sample {
  int s1 = 0;
  Sample mSample1 = new Sample();
  
  public void method() {
    int s2 = 1; // 方法内的局部变量s2 存在于栈中
    Sample mSample2 = new Sample(); // 方法内的引用变量mSample2 存在于栈中，但mSample2指向的对象存在于堆上
  }
}

Sample mSample3 = new Sample(); // mSample3 指向的对象的实体存放在堆上，包括这个对象的所有成员变量s1 和mSample1，而mSample3 它自己存在于栈中。
```

###### 结论

- 局部变量的基本数据类型和引用存储于栈中，引用的对象实体存储于堆中。---- 因为它们属于方法中的变量，生命周期随方法而结束。
- 成员变量全部存储与堆中（包括基本数据类型，引用和引用的对象实体）---- 因为它们属于类，类对象终究是要被new出来使用的。

###### Java如何管理内存

- 使用有向图的方式进行内存管理，可以消除引用循环的问题，这种方式精度很高但效率较低
- 使用计数器（例如COM模型采用计数器方式管理构件），精度较低（很难处理循环引用的问题）但执行效率高

###### 内存泄漏

- 在java中，内存泄漏就是存在一些被分配到对象，这些对象有以下两个特点：首先这些对象是可达的，即在有向图中存在通路可以与其相连；其次，这些对象是无用的，即程序以后不会再使用这些对象。如果对象满足这两个条件，这些对象就可以判定为java中的内存泄漏，这些对象不会被GC所回收，然而它却占用内存。
- 在C++中，内存泄漏的范围更大一些。有些对象被分配了内存空间，然后却不打，由于C++中没有GC，这些内存将永远收不回来。在Java中，这些不可达的对象都由GC负责回收，因此程序员不需要考虑这部分的内存泄漏

![image-20210304192107499](/Users/young1/Library/Application Support/typora-user-images/image-20210304192107499.png)

```java
	// java内存泄漏的典型例子
  Vector v = new Vector(10)
  for (int i = 1; i < 100; i++) {
    Object o = new Object();
    v.add(o); // 将所申请的对象放入一个Vector中
    o = null;  // 仅仅释放引用本身，那么Vector仍然引用该对象，所以这个对象对GC来说是不可回收的，所以存在内存泄漏
  }

// 对象加入Vector后，还必须从Vector中删除，最简单的方法就是将Vector对象设置为null，使有向图不可达，这时GC就可以回收了。
```

###### 引起内存泄漏的原因

总的来讲内存泄漏指的是无用对象持续占有内存或者无用对象得不到及时释放，从而造成内存空间的浪费。

长生命周期的对象持有短生命周期对象的引用就很可能发生内存泄漏，尽管短生命周期对象已经不在需要，但是因为长生命周期持有它的引用而导致不能被回收，这就是java中内存泄漏的发生场景。

具体主要有如下几大类：

1. 静态集合类引起内存泄漏

   ```java
   static Vector v = new Vector(10); // 静态变量的生命周期与应用程序一致，他们所引用的所有的对象Object也不能被释放，因为他们一直被Vector等引用着。
   for (int i = 1; i < 100; i++) {
     Object o = new Object();
     v.add(o);
     o = null; // 仅仅释放了引用本身，Vector仍然引用着对象
   }
   ```

2. 当集合里面的对象属性被修改后，再调用remove（）方法时不起作用

   // 有待验证

3. 监听器

   通常一个应用当中会用到很多监听器，我们会调用一个控件的addxxxListener来增加监听，但往往释放对象的时候却没有记住去删除这些监听器，从而增加了内存泄漏的机会

4. 资源为关闭造成内存泄漏

   对于使用了broadcastreceiver，ContentObserver，File，Cursor，Stream，Bitmap等资源的使用，应该在activity销毁时及时关闭或者注销否则这些资源将不会被回收，造成内存泄漏。对bitmap的回收先调用recycle然后在设置为null，原因：Bitmap对象的内存空间一部分是java的一部分时C的，recycle就是针对C部分的内存释放。

5. 单例造成的内存泄漏

   由于单例的静态特性使得其生命周期跟应用的生命周期一样长，所以如果使用不恰当的话，容易造成内存泄漏。比如持有Activity的context，解决方案：可以持有Application的context

6. 匿名内部类/非静态内部类和异步线程

   非静态内部类创建静态实例造成的内存泄漏

   ```java
   public class MainActivity extends AppCompatActivity {
     private static TestResource mResource = null;
     @Override
     protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.activity_main);
       if (mResource == null) {
         // 静态实例，该实例的生命周期和应用一样长。而这个非静态内部类又会一直持有Activity的引用，导致Activity的内存资源不能正常回收而导致内存泄露
         mResource = new TestResource();
       }
       // ...
     }
     
     // 非静态内部类，默认持有外部类的引用
     class TestResource {
       // ...
     }
   }
   
   // 正确的做法是将非静态内部类改为静态内部类或者将内部类抽取出来封装成一个单例。
   ```

   匿名内部类，如果被异步线程持有了，释放不及时就很容易导致内存泄漏

7. Handler造成的内存泄漏

   由于Handler属于TLS（Thread Local Storage）变量，生命周期和Activity是不一致的。Handler、Message、MessageQueue都是相互关联在一起的，万一Handler发送的Message尚未被处理，则该Message以及发送它的Handler对象将被线程MessageQueue一直持有。

   避免方法：在Activity中避免使用非静态内部类，同时通过弱引用的方式引入Activity，避免直接将Activity座位context传进去。




##### synchronized和volatile的区别

[synchronized和volatile的区别](https://zhuanlan.zhihu.com/p/61966479)



![image-20210324101909999](/Users/young1/Library/Application Support/typora-user-images/image-20210324101909999.png)

###### 基本概念

- 可见性：一个线程对共享变量值的修改，能够及时地被其他线程看到
- 共享变量：如果一个变量在多个线程的工作内存中都存在副本，那么这个变量就是这几个线程的共享变量。

![image-20210324102341321](/Users/young1/Library/Application Support/typora-user-images/image-20210324102341321.png)

线程1对共享变量的修改要想被线程2及时看到，必须要经过如下两个步骤：

1. 把工作内存1中更新过的共享变量刷新到主内存中。
2. 将主内存中更新的共享变量的值更新到工作内存2中。

其中，线程对共享变量的操作，遵循两条规则：

1. 线程对共享变量的所有操作都必须在自己的工作内存中进行，不能直接从主内存中读写。
2. 不同线程之间无法直接访问其他线程工作内存中的变量，线程间变量值的传递需要通过主内存来完成。

###### synchronized实现了可见性和原子性（同步）

JMM关于synchronized的两条规定：

- 线程解锁前，必须把共享内存变量的最新值刷到主内存中
- 线程加锁时，将清空工作内存中共享变量的值，从而使用共享变量时，需要从主内存中重新读取最新的值（注意：加锁与解锁需要时同一把锁）

###### 线程执行互斥代码的过程；

1. 获得互斥锁
2. 清空工作内存
3. 从主存中拷贝变量的最新副本到工作的内存
4. 执行代码
5. 将更改后的共享变量的值刷到主内存
6. 释放互斥锁

###### volatile实现可见性

volatile能够保证变量的可见性，不能保证**变量复合**操作的原子性。是通过加入**内存屏障和禁止重排序优化**来实现的。

private volatile int number = 0;  number++; //不是原子操作

加锁自增，可以保证原子性。

- 对volatile变量执行写操作时，会在写操作后加入一条store屏障指令
- 对volatile变量执行读操作时，会在读操作前加入一条load屏障指令

要在多线程中安全的使用volatile变量，必须同时满足：

- 对变量的写入操作不依赖其当前值（number++， count = count * 5就不满足)
- 该变量没有包含在具有其他变量的不变式中（low < up不满足)

###### 总结

- volatile不需要加锁，比synchronized更轻量级，不会阻塞线程
- 从内存可见角度，volatile读相当于加锁，volatile写相当于解锁
- synchronized既能够保证可见性，又能保证原子性，而volatile只能保证可见性，无法保证原子性

#### 多线程

##### 线程的5个状态：新生状态、就绪状态、运行状态、阻塞状态、死亡状态

- 新建状态（New）：线程对象被创建后就进入了新建状态 Thread thread = new Thread();

- 就绪状态（Runnable）：也称为"可执行状态"。调用了线程对象start()方法后就处于就绪状态，随时可能被CPU调度

- 运行状态（Running）：CPU调度到该线程，该线程进入运行状态。注意：线程只能从就绪状态进入运行状态

- 阻塞状态（Blocked）：线程因为某种原因放弃CPU使用权，暂时停止运行。直到线程进入就绪状态才有机会转到运行状态。

  阻塞分三种情况：

  1. 等待阻塞：通过调用线程wait()方法，让线程等待某工作的完成
  2. 同步状态：线程在获取synchronized同步锁失败（因为锁被其它线程占用），会进入同步阻塞状态
  3. 其它阻塞：通过线程sleep()或者join()或发出I/O请求时，线程会进入到阻塞状态。

- 死亡状态（Dead）：执行完了或者因异常退出了run方法，该线程结束生命周期。

##### 线程中的方法

sleep():sleep持有锁但不会释放锁，但有InterruptException异常抛出

yield():yield礼让，放弃当前的CPU资源，将它让给其他的任务去占用CPU时间。礼让有可能成功有可能不成功要看CPU心情。yield是礼让给同等优先级或更高优先级的线程，不会礼让比自己优先级更低的线程。yield也不会释放锁标记，但有InterruptException异常抛出。

join():强行插入，主要作用是同步。在A线程中调用了B线程的join()方法，如果不传参数或者参数传入0时join(0)表示A等B执行完之后再继续执行。如果是join(10)表示A等待B执行10ms后继续执行。join()必须在start()之后调用，毕竟是插队嘛，有队才能插。

setPriority():设置优先级，在java中线程优先级1-10数字越大优先级越高，超过这个范围会抛出IllegalArgumentException异常。优先级具有继承性：A线程创建B线程，则B线程的优先级和A线程是一样的；优先级具有规则性，高优先级的线程总是大部分先执行完；优先级具有随机性，因为线程的执行具有不确定性和随机性所以并不是优先级高的级 一定先执行，特别是优先级比较接近的时候；

setDaemon(true):守护线程是一种特殊的线程，作用是为其他线程的运行提供便利，GC线程就是典型的守护线程。

关于守护线程：

- thread.setDaemon(true)必须在thread.start()之前设置，否则会抛出一个IllegaleStateException
- 在Daemon线程中产生的新线程也是Daemon的。新建一个线程默认是用户线程
- 守护线程应该永远不去访问固有资源，如文件、数据库，因为它会在任何时候甚至一个操作的中间发生中断
- 守护线程在没有用户线程的情况下会结束（所有用户线程的结束会导致守护线程的结束，进而导致进程的结束）；用户线程只有在线程完结的情况下才会结束。
- 守护线程run执行完毕也会结束线程，并不是说守护线程就一直存活

##### synchronized和ReentrantLock的区别

- synchronized的是隐形锁，reentrantLock（可重入锁）是显性锁
- synchronized不需要用户手动去释放锁，代码执行完后会自动释放，reentrantLock需要手动释放，否则可能会造成死锁现象
- synchronized是不可中断型锁，除非加锁的代码出现异常或者正常执行完成；ReentrantLock可以中断，可通过trylock(long timeout, TimeUnit unit)设置超时方式或将lockInterruptibly()放到代码块中，调用interrupt方法进行中断。
- synchronized为非公平锁，ReentrantLock则既可以选公平锁也可以选非公平锁，通过构造方法new ReentrantLock时传入boolean值进行选择，为空默认false非公平锁，true为公平锁
- synchronized可以锁方法也可以锁代码块，ReentrantLock只能锁代码块
- synchronized不能绑定Condition条件，只能通过Object类的wait()/notify()/notifyAll()方法要么随机唤醒一个线程要么唤醒全部线程；ReentrantLock通过可以绑定condition结合await()/singal()方法实现线程的精准唤醒
- synchronized锁的是对象，锁是保存在对象头里面的，根据对象头数据来标识是否有线程获得锁/争抢锁，要锁住增删改的对象才行；ReentrantLock锁的是线程，根据进入的线程和int类型的state标识来获得/争抢。

##### Looper死循环为什么不会导致应用卡死

应用卡死ANR压根跟这个Looper没有关系，应用在没有消息需要处理的时候它是睡眠，释放线程；卡死是ANR，而Looper是睡眠。

真正的ANR是消息没有及时处理而不是线程等待。而android所有的消息都是message处理的。

主线程等待，让出cpu时间片给其他线程用。

AMS：5s

##### RecyclerView事件拦截机制

- ACTION_DOWN: 当列表在自动滚动的状态下会拦截，用于处理停止滚动。
- ACTION_MOVE: 当手指移动的距离在对方方向上超过了阈值，就会拦截掉事件，用于列表滚动。
- ACTION_UP: 根据当前列表是否处于滚动状态选择是否拦截。

ListView原生控件有setOnItemClickListener而RecyclerView原生则没有。ListView的setOnItemClickListener()方法注册的是子项的点击事件，但如果想点击子项里具体但某一个按钮处理起来就比较麻烦。RecyclerView干脆直接摒弃了子项点击事件的监听器，让所有的点击事件都由具体的View去注册。

自定义的onItemTouchListener的拦截过程：

1. 我们自定义实现onItemTouchListener,需要通过addOnItemTouchListener()添加到RecyclerView里。
2. RecyclerView在判断拦截事件时，会优先判断有没有自定义的onItemTouchListener要拦截此事件，如果有，则会帮他拦截下来。
3. RecyclerView在处理事件时，也会优先判断有没有自定义的onItemTouchListener要处理该次事件，如果有就交给他处理，自己不再处理。
4. 

##### RecyclerView缓存机制

RecyclerView根据不同状态可分为：屏幕内缓存、屏幕外缓存、自定义缓存、缓存池。RecyclerView是通过内部类Recycler来管理缓存。

###### 一级缓存：屏幕内缓存（mAttachedScrap）

屏幕内缓存指在屏幕中显示的ViewHodler，这些ViewHolder会缓存在mAttachedScrap、mChangedScrap中：

- mChangedeScrap表示数据已经改变的ViewHolder列表，需要重新绑定数据（调用onBindViewHolder)
- mAttachedScrap表示未与RecyclerView分离的ViewHolder列表

###### 二级缓存：屏幕外缓存（mCachedViews)

用来缓存一出屏幕之外的ViewHolder，默认情况下缓存容量是2，可以通过setViewCacheSize方法来改变缓存的容量大小。如果mCachedViews容量已满，则会优先移除旧ViewHolder，把旧的ViewHolder移入到缓存池RecycleredViewPool中。

###### 三级缓存：自定义缓存（ViewCacheExtension）

给用户的自定义扩展缓存，需要用户自己管理View的创建和缓存，可通过RecyclerView.setViewCacheExtension()设置。

###### 四级缓存：缓存池（RecycledViewPool）

ViewHolder缓存池，在mCachedView中如果缓存已满的时候（默认最大值为2），先把mCahcedViews中旧的ViewHolder存入到RecyclerViewPool。如果RecyclerViewPool缓存池已满，就不会再缓存。从缓存池中取出ViewHolder，需要重新调用bindViewHolder绑定数据。

- 按照viewType来查找ViewHolder
- 每个ViewType默认最多缓存5个
- 可以多个RecyclerView共享RecycledViewPool

###### RecyclerView缓存策略

Recyclerview在获取ViewHolder时按照四级缓存的顺序查找，如果没找到就创建。其中只有RecycledViewPoo找到时才会调用onBindViewHolder,其它缓存不会重新bindViewHolder。

mAttachedScrap --> mCachedViews --> mViewCacheExtension --> RecycledViewPool

###### RecyclerView优化

- 降低item的布局层次：可以减少界面创建的渲染时间，使用约束布局等。
- 去除冗余的setOnItemClick事件：直接在onBindViewHolder方法中创建一个匿名内部类的方式来实现setOnItemClikck是不可取的，这会导致RecyclerView快速滑动时创建很多对象，优化方法：事件的绑定和viewholder对应的rootView进行绑定。
- 复用pool缓存：如果存在RecyclerView中嵌套RecyclerView的情况，可以考虑复用RecyclerViewPool缓存池，减少开销。

RecyclerView最多可以缓存N（屏幕最多可显示的item数）+2（屏幕外的缓存）+5*M（M代表M个viewType，缓存池的缓存）。





##### 什么是冷启动、温启动、热启动

冷启动：App完全被杀死了，这个时候去启动它就是冷启动。

温启动：如果App的主Activity被销毁或者回收了，这个时候去启动它就是温启动。

热启动：如果App只是被挂起到了后台，这个时候去启动它就是热启动。

冷启动开始时，系统有三个任务：加载并启动应用；在启动后立即显示应用的空白启动窗口；创建应用进程。

系统统一创建应用进程，应用进程就负责后续阶段：

1. 创建应用对象。
2. 启动主线程。
3. 创建主activity。
4. 扩充试图。
5. 布局屏幕。
6. 执行初始绘制。

一旦应用进程完成第一次绘制，系统进程就会换掉当前显示的后台窗口，替换为主activity。此时，用户可以开始使用应用。



##### sdk自动初始化

可以通过ContentProvider完成自动初始化,因为ContentProvider是在Application.onCreate之前创建的。

```java
		<application>
        <provider
            android:name="me.jessyan.autosize.InitProvider"
            android:authorities="${applicationId}.autosize-init-provider"
            android:exported="false"
            android:multiprocess="true"/>
    </application>
              
 public class InitProvider extends ContentProvider {
    @Override
    public boolean onCreate() {
        Context application = getContext().getApplicationContext();
        if (application == null) {
            application = AutoSizeUtils.getApplicationByReflect();
        }
        AutoSizeConfig.getInstance()
                .setLog(true)
                .init((Application) application)
                .setUseDeviceSize(false);
        return true;
    }
 }             
              
AndroidManifest中注册provider：InitProvider， contentProvider会在Application.OnCreate之前创建，因此可以在provider的onCreate里初始化
```



##### ScrollView嵌套ListView的时候为什么显示不全

ScrollView和LisetView都继承于ViewGroup，在绘制的时候都会调用onMeasure方法进行测量。

MeasureSpec.getMode可以获取宽高模式：

- MeasureSpec.AT_MOST 相当于wrap_content
- MeasureSpec.EXACTLY 相当于具体数值100dp，match_parent, fill_parent
- MeasureSpec.UNSPECIFIED 尽可能大，ScrollView ListView 就是这个模式

ListView的onMeasure方法里通过MeasureSpec.getMode判断是否是UNSPECIFIED，如果是只给一个item的大小，解决方案：重写ListView的onMeasure方法

```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
  heightMeasureSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE>>2, MeasureSpace.AT_MOST);
  super.onMeasure(widthMeasureSpec, heightMeasureSpec);
}
```



##### View的渲染逻辑

Android系统每隔16ms（人眼与大脑之间的协作无法感知超过60fps的画面更新）就会重绘一次activity，也就是说，我没得应用必须在16ms内完成屏幕刷新的全部逻辑操作，即每一帧只能停留16ms。



##### ViewModel为什么横竖屏转换时为什么重建。

Activity里有创建viewModel是通过ViewModelProvider（viewModelStore，viewmodelProviderFactory）.get(viewModelClass)来获取的。ViewModelStore 用HashMap<String, ViewModel>来管理ViewModel的，key为ViewModel的class名，value就是要保存的viewModel。横竖屏切换在onDestroy之前 会将viewModelStore保存在nonConfigurationInstances里，而在attach里会获取lastNonConfigurationInstances对象，在获取ViewModelStore的时候会先判断lastNonConfigurationInstance是否为空，不为空才从lastConfigurationInstance里获取viewModelStore。

activity在onDestroy的时候会调用viewModelStore的clear方法，从而将viewModel移除。

##### SQLite

```java
// 定义一个类继承SQLiteOpenHelper
public class DBHelper extends SQLiteOpenHelper {
  // 带有全部参数的构造函数，此构造函数是必须需要的。
	public DBHelper(Context context, String name, CursorFactory factory, int version) {
    super(context, name, factory, version);
  }
  
  @Override
  public void onCreate(SQLiteDatabase db) {
    // 创建数据库sql语句
    String sql = "create table user(name verchar(20))";
    // 执行sql语句
    db.execSQL(sql);
  }
  
  	@Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }
}


```









##### 为什么不能使用Application Context 显示dialog















## 四、经验问题

#### 1. .9图制作及注意事项

- 制作.9的图片不要压缩，压缩后制作出的.9图有问题，会导致编译不通过
- 制作出的.9图要放到drawable文件夹下，命名为xxx.9.png
- 找到制作好的.9图，打开选到9_patch进行四条边的缩放范围设定，右下角有对应的x y坐标
- android studio 右键-->create 9-patch file-->save
- 另外作为bacground，如果图片切图比较大，即时.9图也不会缩到很小。解决方案：可以利用布局+imageview scaleType的fitXY来设置。Relative有alignLeft alignRight。
- .9图片当 横竖都可以拉伸时，如果宽/高一边小于切图，另外一遍大于切图，则显示出来依旧是变形的。如果两边都小于切图且比例相同或者两边都大于切图，则显示正常
- 



#### 2.Fragment onCreateView / onViewCreated

onCreateView 方法是在UI线程，onViewCreated未必执行在UI线程。表象：switch控件，设置初始值，在onCreateView里直接设置生效，在onViewCreated里需要在view.post里去设置才会完全生效。

