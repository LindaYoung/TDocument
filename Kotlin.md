### Kotlin

#### 1. Kotlin基础语法（与java不同的地方）

##### 常用操作符--下标操作类

- contains - 判断是否有指定元素
- elementAt - 返回对应元素，越界会抛IndexOutOfBoundsException
- firstOrNull - 返回符合条件的第一个元素，没有 返回null
- lastOrNull - 返回符合条件的最后一个元素，没有 返回null
- indexOf - 返回制定元素的下标， 没有 返回-1
- singleOrNull - 返回符合条件的单个元素，如果没有符合或超过一个 返回null

##### 常用操作符 - 判断类

- any - 判断集合中是否有满足条件的元素
- all - 判断集合中的元素 是否都满足条件
- none - 判断集合中是否 都不满足条件，是 则返回true
- count - 查询结合中 满足条件 的 元素的个数
- reduce - 从 第一项到最后一项进行累计（相加）

##### 常用操作符 - 过滤类

val list = listOf(1,2,3,4,5,6)

- filter - 过滤所有满足条件的元素 	     list.filter{it % 2 == 0} 	// [2, 4, 6]
- filterNot - 过滤掉所有不满足条件的元素   list.filterNot{it %2 == 0} // [1,3,5]
- filterNotNull - 过滤掉Null                           list.filterNotNull(). // [1,2,3,4,5,6]
- take - 返回前n个元素                               list.take(2) // [1,2]

##### 常用操作符 - 转换类

val list = listOf(1,2,3,4,5,6)

- map - 转换成另一个集合                                                 list.map{it * 2} // [2,4,6,8,10,12]
- mapIndexed - 转换成另一个集合并可以拿到Index.    list.mapIndexed {index, it -> index * it} // [0, 2,6,12,20,30]
- mapNotNull - 执行转换前过滤掉为null的元素             list.mapNotNull{it * 2} // [2,4,6,8,10,12]
- flatMap - 自定义逻辑合并两个集合                               list.flatMap{listOf{it, it + 1} //[1,2,2,3,3,4,4,5,5,6,6,7]
- groupBy - 按照某个分组，返回Map                             list.groupBy{if(it % 2 == 0) "even" else "odd"} //{odd=[1,3,5], even=[2,4,6]}

##### 常用操作符 - 排序类

- reversed - 反序（倒序）
- sorted - 升序
- sortedBy - 自定义排序
- sortedDescending - 降序



##### inline、 noinline、 crossinline区别

- inline ：内联，在编译期间起作用。编译器在编译的时候把这个函数（方法以及方法中lambda参数）复制到调用处。（减少函数调用次数；减少对象的生成；）
- No inline:修饰的是inline方法中lambda参数。noinline用于外面不想让inline特性作用到inline方法的某些lambda参数上的场景。
- crossinline：保留了inline特性，但如果想在传入的lambda里面return的话，就会报错。return只能return当前的这个lambda。

总结：`inline` 关键字的作用，是把 `inline` 方法以及方法中的 `lambda` 参数在编译期间复制到调用方，进而减少函数调用以及对象生成。对于有时候我们不想让 `inline` 关键字对 `lambda` 参数产生影响，可以使用 `noline` 关键字。如果想 `lambda` 也被 `inline`，但是不影响调用方的控制流程，那么就要是用 `crossinline`。

#### 2. Kotlin的类与对象



#### 3.Kotlin的集合



#### 4. 高阶函数与Lambda



#### 5.Kotlin泛型与注解



#### 6.Kotlin写成框架基础



#### 7.Kotlin与java互调原理项目实践



#### 8.Kotlin协程原理分析项目实践

##### 	a. 协程是什么

​		协程是一个广义的概念，很多编程语言里都有协程的编程思想。协程是一种在程序中处理并发任务的方案。和线程属于同一个层级的概念，是一种和线程不同的并发任务解决方案。

​		kotlin协程和广义的协程不是同一个东西，kotlin协程（kotlin for jvm）是一个线程框架，类似于java重的Executor。

##### 	b .协程优势

​		用起来方便，不需要回调，线程切换方便。可以保证所有的耗时操作一定放后台执行（自己写代码注意切换即可）

##### 		c.suspend关键字

​		suspend并不是用来切换线程的，实际的切换线程操作是由其它函数完成，比如withContext

​		suspend主要是起到标记和提醒的作用。语法层面：通过报错来提醒调用者和编译器，这是一个耗时函数，需要放在后台执行；编译器层面：辅助kotlin编译器来把代码转换成JVM字节码。

##### 	d.挂起

​		本质其实是结束了当前的线程，然后切换到其它线程去执行，withcontext执行完后可以回到原本的线程。



##### 	A. runBlocking , launch, withContext的区别

```kotlin
// 通过Coroutine.launch开启一个协程，协程体里的任务会先suspend，让Coroutine.launch后面的代码继续执行，直到协程体内的方法执行完成在自动切回来
fun testCoroutine() {
        // launch 是阻塞的 runBlocking是非阻塞的
        // 结果： 2，0，1 main
        CoroutineScope(Dispatchers.Main).launch {
            printMsg("当前线程为0： ${Thread.currentThread().name}")
            delay(1000) //suspend 挂起函数
            printMsg("当前线程为1： ${Thread.currentThread().name}")
        }
        printMsg("当前线程为2： ${Thread.currentThread().name}")
    }

// runBlocking里的任务如果是非常耗时的操作时，会一直阻塞当前线程，在实际开发中很少用到runBlocking。由于runBlocking接收的lambda代表着一个coroutineScope，所以runBlocking协程体内可继续通过launch来继续创建一个协程，比米娜了lauch所在的线程已经运行结束而切不回来的情况。
fun testCoroutine() {
        // launch 是阻塞的 runBlocking是非阻塞的
        // 结果： 2， 3， 4， 5， 0， 1 main
        CoroutineScope(Dispatchers.Main).launch {
            printMsg("当前线程为0： ${Thread.currentThread().name}")
            delay(1000) //suspend 挂起函数
            printMsg("当前线程为1： ${Thread.currentThread().name}")
        }
        printMsg("当前线程为2： ${Thread.currentThread().name}")

        // runBlocking 非阻塞的。
        runBlocking {
            printMsg("当前线程为3： ${Thread.currentThread().name}")
            delay(1000)
            printMsg("当前线程为4： ${Thread.currentThread().name}")
        }
        printMsg("当前线程为5： ${Thread.currentThread().name}")
    }
```

###### 可返回结果的协程witchContext和async

withContext和async都可以返回耗时任务的执行结果。一般来说，多个withContext的任务时串行的，且withContext可直接返回耗时任务的结果。多个async任务时并行的，async返回的事一个Deferred<T>，需要调用其await()方法获取结果。

Await() 挂起函数，但实际上只有在async未执行完成返回结果是，才会挂起协程；若async已经有结果了，await则直接获取其结果并复制给变量，此时不会挂起协程。



##### 基础

1. 什么是进程？为什么要有进程？

   进程是资源分配的最小单位。

   挂起：保存程序当前状态，暂停当前程序；激活：恢复程序状态，继续执行程序。

   进程是一个具有一定独立功能的程序在一个数据集上的一次动态执行的过程，是操作系统进行资源分配和调度的一个独立单位，是应用程序运行的载体。

   操作系统之所以要支持多进程，是为了提高CPU的利用率而为了切换进程，需要进程支持挂起和恢复，不同进程间需要的资源不同，所以这也是为什么进程间需要隔离，这也是进程是资源分配的最小单位的原因。

   

2. 什么是线程？为什么要有线程？

   线程是CPU调度的最小单元。

   线程的出现是为了降低上下文切换消耗，提高系统的并发性，并突破一个进程只能干一件事的缺陷，使得进程内并发成为可能。

   进程和线程都是一个时间段段描述，是CPU工作时间段段描述，只是颗粒大小不同。

   

3. 什么是协作式？什么是抢占式？

   协作式：由进程主动让出执行权。

   抢占式：由操作系统决定执行权，操作系统具有从任何一个进程取走控制权和使另一个进程获得控制权的能力。（时间片轮转调度）

   

4. 为什么要引入协程？是为了解决什么问题？

   协程的核心竞争力在于：它能简化异步并发任务，以同步方式写异步代码。

   线程和协程最大的区别在于：线程是被动挂起恢复，协程是主动挂起恢复。

   

   a . 协程根据是否开辟相应的函数调用栈分为两类：有栈协程和无栈协程

   - 有栈协程：有自己的调用栈，可在任意函数调用层级挂起，并转移调度权。
   - 无栈协程：没有自己的调用栈，挂起点点状态通过状态机或闭包等语法来实现。

   b. suspend挂起函数

   suspend 的本质就是CallBack, suspend函数不能在协程体外调用，原因是CPS转换的时候会有Continuation实例传递。

   只要被suspend修饰的函数都是挂起函数，但不是所有的挂起函数都会被挂起，只有当挂起函数里包含异步操作时，它才会被真正挂起。

   ```kotlin
   suspend fun getUserInfo(): String {
     withContext(Dispatcher.IO) {
       delay(1000L)
     }
     return "BoyCoder"
   }
   
   // 反编译结果
   public static final Object getUserInfo(Continuation $conpletion) {
     ...
     return "BoyCoder";
   }
   
   public interface Continuation<in T> {
     // 通过context获取上下文资源，保存挂起时的一些状态和资源。coroutineContext主要承载了资源获取，配置管理等工作，是执行环境相关的通用数据资源的统一提供者
     public val context: CoroutineContext
     
     // 相当于onSuccess 结果
     public fun resumeWith(result: Result<T>)
   }
   ```

   CPS 转换（Continuation - Passing - Style Transformation），CPS其实就是将程序接下来要执行的代码进行传递的一种模式。CPS转换，就是将原本的同步挂起函数转换成CallBack异步代码的过程。这个转换是编译器在背后做的，我们程序员对此无感知。

5. 状态机

   kotlin协程的实现依赖于状态机。

   - 协程实现的核心就是CPS变换与状态机
   - 协程执行到挂起函数，一个函数如果被挂起了，它的返回值会是CoroutineSingletons.COROUTINE_SUSPEND
   - 挂起函数执行完后，通过Continuation.resume方法回调，这里的Continuation是通过CPS传入的
   - 传入的Continuation实际上是ContiunationImpl, resume方法最后会再次回调到invokeSuspend方法中
   - invokeSuspend方法即是我们写的代码执行的地方，在协程运行过程中会执行多次
   - invokeSuspend中通过状态机实现状态的流转
   - continuation.label是状态流转的关键，label改变一次代表协程发生了一次挂起恢复
   - 通过break label实现goTo的跳转效果
   - 我们写在协程里的代码，被分到状态机里各个状态中，分开执行
   - 每次协程切换后，都会检查是否发生异常
   - 切换协程之前，状态机会把之前的结果以成员变量的方式保存在continuation中



##### 线程总结

- 线程是操作系统级别的概念
- 我们开发者通过编程语言(Thread.java)创建的线程，本质还是操作系统内核线程的映射
- JVM中的线程与内核线程 存在映射关系，有一对一，一对多，多对多。JVM在不同操作系统中的具体实现会有差别，一对一是主流。
- 一般情况下，我们说的线程都是内核线程，线程之间的切换，调度，都由操作系统负责
- 线程也会消耗操作系统资源，但比进程轻量很多
- 线程，是抢占式的，它们之间能共享内存资源，进程不行
- 线程共享资源导致了多线程同步问题
- 有的编程语言会自己实现一套线程库，从而能在一个内核线程中实现多线程效果，早起JVM的“绿色线程”就是这么做的，这种线程被称为“用户线程”。

##### Kotlin协程总结：

- kotlin协程，不是操作系统级别的概念，无需操作系统支持
- kotlin协程，有点像上面提到的“绿色线程”，一个线程上可以运行成千上万个协程
- kotlin协程，是用户太的(userlevel)，内核对协程无感知
- kotlin协程，是协作式的，由开发者管理，不需要操作系统进行调度和切换，也没有抢占式的消耗，因此它更加高效
- kotlin协程，它底层基于状态机实现，多协程之间共用一个实例，资源开销极小，因此它更加轻量
- kotlin协程，本质还是运行于线程智商，它通过协程调度器，可以运行到不同的线程上

[参考链接1](https://juejin.cn/post/6883652600462327821?share_token=c69290f1-92e0-46c0-8ff0-ce7a274c5643)

[参考链接2](https://juejin.cn/post/6973650934664527885)



#### 9.Kotlin语法糖总结

- also（内联扩展函数）：also函数的结构实际上和let很像，唯一的区别就是返回值不一样。let是以闭包返回，返回函数体内最后一行的值。而also函数返回的则是传入对象的本身。一般可用于多个扩展函数链式调用
- with（不是内联拓展函数）：适用于同一个类的多个方法时，可以省去类名重复，直接调用类的方法即可。经常用于Android中RecyclerView中onBinderViewHolder中，数据model的属性映射到UI上
- run（内联扩展函数）：适用于let，with函数任何场景，因为run函数是let，with两个函数的结合体。它弥补了let函数在函数体内必需使用it参数替代对象，在run函数中可以想with函数一样可以省略，直接访问实例的公有属性和方法，另一方面它弥补了with函数传入对象判空问题，在run函数中可以像let函数一样做判空处理。
- apply（内联扩展函数 inline+lambda）：和run函数很像，唯一不同点就是各自返回值不一样，run函数是以闭包形式返回最后一行代码的值，而apply函数的返回的是传入对象的本身。apply一般用于一个对象实例初始化的时候，需要对对象中的属性进行赋值。或者动态inflate出一个xml的view的时候需要给view绑定数据也会用到。



