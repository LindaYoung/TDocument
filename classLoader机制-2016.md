***\*ClassLoader 机制\****

 

1.定义：

Java虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校检、转换解析和初始化的，最终形成可以被虚拟机直接使用的Java类型，这就是虚拟机的类加载机制。

 

2.ClassLoader加载类的过程：

  ① 根据一个类的全限定名来获取定义此类的二进制字节流

② 将这个字节流所代表的静态存储结构转化为JVM方法区中的运行时数据结构

③ 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口。

 

3.插件加载机制：

宿主每打开一个应用，都会另新开一个进程，重新去hook一遍系统环境，也会重新生成一个这个应用对应的ClassLoader对象。利用这个ClassLoader对象去动态加载相应的类。

 

4.插件的ClassLoader对象的生成：

① 系统Activity的创建过程：系统通过待启动的Activity的类名className，然后使用ClassLoader对象cl把这个类加载进虚拟机，最后使用反射创建了这个Activity类的实例对象。代码如下：

java.lang.ClassLoader cl = r.packageInfo.getClassLoader();

activity = mInstrumentation.newActivity(cl, component.getClassName(), r.intent);

StrictMode.incrementExpectedActivityCount(activity.getClass());

r.intent.setExtrasClassLoader(cl);

 

② 获取 r.packageInfo对象：其实系统直接调用了getPackageInfo方法得到这个对象。在这个方法里面，先去获取缓存数据；如果没有命中缓存数据，才通过LoadedApk的构造函数创建了LoadedApk对象，也就是r.packageInfo对象；

③ 通过各种反射根据apk文件去生成LoadedApk对象，然后将这个对象存入系统缓存中，这样，系统加载类时，就会使用我们添加进去的LoadedApk的ClassLoader来加载这个特定的类。

 

5.ClassLoader机制 详细讲解地址：http://weishu.me/2016/04/05/understand-plugin-framework-classloader/

