### MVC，Model-View-Controller

- Model, 模型层，即数据模型，用于获取和存储数据。
- View，视图层，即xml布局。
- Controller，控制层，负责业务逻辑。

![image-20210421162016485](/Users/young1/Library/Application Support/typora-user-images/image-20210421162016485.png)

View层接收到用户操作事件，通知到Controller进行对应逻辑处理，然后通知Model去获取/更新数据，Model再把更新的数据通知View更新界面。但在Android中，因为xml布局能力很弱，View的很多操作时在Activity/Fragment中的，而业务逻辑同样也卸载Activity/Fragment中。

MVC问题点：

1. Activity/Fragment责任不明，同时负责View、Controller，导致其中代码量大不满足单一原则。
2. Model耦合View，View的修改会导致Controller和Model都进行改动，不满足最少知识原则。



### MVP，Model-View-Presenter

- Model，模型层，即数据模型，用于获取和存储数据。
- View，视图层，即Activity/Fragment
- Presenter，控制层，负责业务逻辑

![image-20210421162550780](/Users/young1/Library/Application Support/typora-user-images/image-20210421162550780.png)

View层接收到用户操作事件，通知Presenter，Presenter进行逻辑处理，然后通知Model更新数据，Model把更新的数据给到Presenter，Presenter再通知View更新洁面。

实现思路

- UI逻辑抽象成IView接口，由具体的Activity实现类完成。且调用Presenter进行逻辑操作。
- 业务逻辑抽象成IPresenter接口，由具体的Presenter实现类来实现。逻辑操作完成后调用IView接口方法刷新UI

MVP本质时面向接口编程，实现类以来倒置原则。MVP解决了View层责任不明的问题，但没解决代码耦合的问题，View和Presenter之间相互持有

MVP问题点：

1. 会引入大量的IView和IPresenter接口，增加实现类的复杂度
2. View和Presenter相互持有，形成耦合



### MVVM，Model-View-ViewModel

- Model, 模型层，即数据模型，用于获取和存储数据
- View，试图，即Activity/Fragment
- ViewModel，视图模型，负责业务逻辑（对应MVP里的presenter）

Jetpack ViewModel组件是对MVVM的ViewModel的具体实施方案。

MVVM的本质时数据驱动，把解偶做的更彻底，ViewModel不持有view。

![image-20210421163557843](/Users/young1/Library/Application Support/typora-user-images/image-20210421163557843.png)



View产生事件，使用ViewModel进行逻辑处理后，通知Model更新数据，Model把更新的数据给ViewModel，ViewModel自动通知给view更新洁面，而不是主动调用view的方法。



### MVVM的实现 - Jetpack MVVM

优点：不经通过数据驱动完成彻底解偶，还兼顾了Android页面开发中其它不可预期的错误，例如Lifecycle能在妥善处理 页面生命周期 避免view空指针问题，ViewModel是的UI发生重建时无需重新向后台请求数据，节省了开销，让试图重建时更快展示数据。

![image-20210422102535046](/Users/young1/Library/Application Support/typora-user-images/image-20210422102535046.png)

各模块对应MVVM架构：

- View层：Activity/Fragment
- ViewModel层：Jectpack ViewModel+Jectpack LiveData
- Model层：Respository仓库，包含 本地持久性数据 和 服务端数据

图中的所有肩头都是单向的，例如view层指向了ViewModel层，表示View层灰持有ViewModel层的引用，但反过来ViewModel层却不能持有View层的引用，初次之外，引用也不能跨层持有，比如View层不能持有仓库层的引用，进击每一层的组件都只能与它相印的组件进行交互。



### MVP改造MVVM

1. 去除Presenter对View、context的引用
2. Presener替换成ViewModel的实现，获取的数据以LiveData呈现
3. 删除定义的IView等接口，Activity/Fragment中获取ViewModel实例，调用其方法获取数据
4. Activity/Fragment观察需要的LiveData然后刷新UI
5. 检查下原MVP的Model层的实现，是否满足要求



[Android 架构探索](https://docs.qq.com/doc/DR0dMQkxpSWNNRU5a)