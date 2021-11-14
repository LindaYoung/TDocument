### 概览

![image-20210820111407624](/Users/young1/Library/Application Support/typora-user-images/image-20210820111407624.png)

注意： 每个组建仅依赖于其下一级的组件，其中 存储区是唯一依赖于其它多个类的类。在本例中，存储区依赖于持久性数据模型和远程后端的数据源。

这种设计打造了一致且愉快的用户体验。无论用户上次使用应用是在几分钟前还是在几天前，现在回到应用时都会立即看到 应用在本地保留的用户信息。如果此数据已过时，则应用的存储区模块将开始在后台更新数据。

### 构建界面

ViewModel对象为特定的界面组件（如Fragment或Activity）提供数据，并包含数据处理业务逻辑，以与模型进行通信。例如，ViewModel可以调用其他组件来加载数据，还可以转发用户请求来修改数据。ViewModel无法感知界面组件，因此不受配置更改（如在旋转设备时重新创建Activity）的影响。





#### LifeCycle

通过观察者模式实现对页面生命周期的监听。系统提供两个类LifecycleOwner和LifecycleObserver。

查看源码ComponentActivity实现了LifecycleOwner接口。自己实现一个自定义组件实现LifecycleObserver，并用@OnLifecycleEvent(Lifecycle.Event.ON_RESUME)来进行监听，在activity里通过getLifecycle().addObserver(自定义类对象)进行绑定。当Activity生命周期发生变化会通知通过LifecycleOwner来通知LifecycleObserver。



#### LiveData



#### ViewModel

##### 简单使用

1. 写一个继承viewModel的类

   ```kotlin
   class ViewModelT : ViewModel() {
       var number: Int = 1
   
       fun plus() {
           number++
       }
   }
   ```

2. A

   ```kotlin
   private fun testViewModel() {
       // ViewModelT.javaClass 提示不能解析javaclass  要用ViewModelT::class.java
     // Activity本身实现了ViewModelStoreOwner
       viewModel = ViewModelProvider(this, ViewModelProvider.AndroidViewModelFactory(application)).get(ViewModelT::class.java)
       textView.text = viewModel.number.toString()
   
       button.setOnClickListener {
           plusNum()
           i++
       }
   }
   
   private fun plusNum() {
           viewModel.plus()
           textView.text = viewModel.number.toString()
       }
   ```

3. d 









### 听课笔记

#### 为什么使用Jectpack

- 遵循最佳做法，可以减少崩溃和内存泄漏
- 消除样板代码
- 减少不一致

#### LifeCycle ---- 生命周期管理，解耦页面与组件







快捷：

1. 怎么直接变成全局变量
2. 界面化添加依赖：上面图标（projectStructure --> Dependencies --> 选择要添加到哪个module里 --> 添加library dependency --> 输入 anroidx.lifecycle 搜索 --> 点击OK
3. 新建一个工程，并用git管理