## 一、自定义view

### 1、 自定义view构造函数写法

```kotlin
class DlLiveRelayMsgView : FrameLayout {
    constructor(context: Context): this(context, null)
    constructor(context: Context, attributeSet: AttributeSet?): this (context, attributeSet, 0)
    constructor(context: Context, attributeSet: AttributeSet?, defStyleAttr: Int): super(context, attributeSet, defStyleAttr)
    init {
        
    }
}
```



```kotlin
class DlLiveRelayMsgView @JvmOverloads constructor(context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0) : FrameLayout(context, attrs, defStyleAttr)
```

这一行抵得上上面的那么多行：注解@JvmOverloads在有默认参数值的方法中使用该注解，Kotlin就会暴露多个重载方法；参数里赋默认值，保证java里调用的时候可以用任意个参数



### 2、外部类实现OnClickListener,内部类注册监听

```kotlin
class TestDemo : OnClickListener {
    inner class ClickTest {
        private val textView = TextView(AppInfo.getContext())
        private fun init() {
            textView.setOnClickListener { v: View -> onClick(v) } //注意是大括号包裹
          	//也可以这样写
            textView.setOnClickListener(this@TestDemo)
        }
    }

    override fun onClick(v: View) {}
}
```

###  3、常量和静态类

##### 静态常量

```java
// java 静态变量
class test {
	public static final String LOAN_TYPE = "loanType";
	public static final String LOAN_TITLE = "loanTitle";
}
```

```kotlin
// kotlin 静态变量
class test {
  companion object {
    val LOAN_TYPE = "loanType";
    val LOAN_TITLE = "loanTitle";
  }
}

// 或者加上StaticParams
class test {
  companion object StaticParams {
    val LOAN_TYPE = "loanType";
    val LOAN_TITLE = "loanTitle";
  }
}

// 或者 加上const 常量修饰，const只能修饰val
class test {
  companion object {
    const val LOAN_TYPE = "loanType";
    const val LOAN_TITLE = "loanTitle";
  }
}
```

```java
// java中对应的引用
class TestEntity {
  public TestEntity() {
    String title = Test.Companion.getLOAN_TITLE();
  }
}

class TestEntity {
  public TestEntity() {
    String title = Test.StaticParams.getLOATION_TITLE();
  }
}

class TestEntity {
  public TestEntity() {
    String title = Test.LOAD_TITLE;
  }
}
```

##### 静态方法

```kotlin
class StaticDemo {
  companion object {
    fun test() {}
  }
}

// 或者
class StaticDemo {
  companion object StaticParams {
    fun test() {}
  }
}
```

```java
// 对应java中的引用
class TestEntity {
  public TestEntity() {
    TestDemo.Companion.test();
  }
}
// 或者
class TestEntity {
  public TestEntity() {
    StaticDemo.StaticParams.test();
  }
}
```

### 4. 小点优化写法

```kotlin
time ?: "上午" // 如果time为null，默认为“上午”
time ?: 0 < 3 // 如果 time.length为空，则直接赋值0，如果不为空，判断字符的个数
time ?.let{ 
  // time不为空执行
}
time ?. length // 如果time!=null 返回time.length
```





###### 初用kotlin时，小记录

1. 内部类声明要用inner 修饰符，否则即使在class A里面声明的class B，B类依旧访问不到A类的成员变量。inner class B表示B是A的内部类，就可以访问了。
2. 找view时，不用通过findviewbyid去找，直接 val relayMsgTip: TextView = itemView.tv_relay_tip
3. init调用顺序：类中声明的属性->init方法->构造函数；如果有伴生对象：伴生对象里的init->init->构造函数，[参考链接](https://blog.csdn.net/yuzhiqiang_1993/article/details/87863589)
4. Object 在kotlin中可以是一种对象表达式也可以是一种对象声明。继承一个匿名对象时是对象表达式；修饰类时表示该类为静态类。
5. :: 表示函数引用的对象，指向函数对象的引用而不是原函数

