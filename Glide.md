[官网](https://muyangmin.github.io/glide-docs-cn/doc/resourcereuse.html)

#### Android图片加载性能的两个关键方面

1. 图片解码速度
2. 解码图片带来的资源压力

##### Glide是用了多个步骤来确保Android上加载图片尽可能的快速和平滑

1. 自动、智能地采样(downsampling)和缓存(caching)，以最小化存储开销和解码次数
2. 积极的资源重用，例如字节数组和Bitmap，以最小化昂贵的垃圾回收和堆碎片化影响
3. 深度的生命周期集成，以确保仅优先处理活跃的Fragment和Activity的请求，并有利于应用在必要时释放资源以避免在后台时被杀掉



#### Glide里的缓存

默认情况下，Glide会在开始 一个新的图片请求之前检查以下多级的缓存：

1. 活动资源(Activit Resources) - 现在是否有另一个view正在展示这张图片？

2. 内存缓存(Memory cache) - 该图片是否最近杯加载过并仍存在于内存中？

3. 资源类型(Resource) - 该图片是否之前曾被解码、转换并写入过磁盘缓存？

4. 数据来源(Data) - 构建这个图片的资源是否之前曾被写入过文件缓存？

前两步检查图片是否在内存中，如果在则直接返回图片。后两步检查图片是否在磁盘上，以便快速但异步地返回图片。

如果四个步骤都未能找到图片，则Glide会返回到原始资源以取回数据（原始文件，url等）。



##### 缓存键（cache keys）

在Glide 4里，所有的缓存键都包含至少两个元素：

1. 请求加载的model（file， UrI， Url）。如果你使用自定义的model，需要正确地实现hashCode()和equal()
2. 一个可选的签名（signature）

另外，上处步骤1～3(活动资源、内存缓存、资源磁盘缓存)的缓存键还包含一些其他数据，包括

1. 宽度和高度
2. 可选的 变换(Transformation)
3. 额外的数据类型 选项(Options)
4. 请求的数据类型(Bitmap, GIF, 或其他)

活动资源和内存缓存使用的键还和磁盘资源缓存策略有不同，以适应哪行选项(Options),比如影响Bitmap配置的选项或者其他解码时才会用到的参数。为了生成磁盘缓存上的缓存键名称，以上的每个元素会被哈希化以创建一个单独的字符串键名，并在随后作为磁盘缓存上的文件名称使用。



#### 配置缓存

Glide提供一系列的选项，以允许你选择加载请求与Glide缓存如何交互

##### 磁盘缓存策略（Disk Cache Strategy）

DiskCacheStrategy可被diskCacheStrate方法应用到每一个单独的请求。目前支持的策略允许你阻止加载过程使用或写入磁盘缓存，选择性地仅缓存无修改的原生数据，或仅缓存变换过的缩略图，或是兼而有之。

默认的策略叫做AUTOMATIC，它会尝试对本地和远程图片使用最佳的策略。当你加载数据时，AUTOMATIC策略仅会存储未被你的加载过程修改过的原始数据，因为下载远程数据相比调整磁盘上已经存菜的数据要昂贵的多。对于本地数据，AUTOMATIC 策略则会仅存储变换过的缩略图，因为即使你需要再次生成另一个尺寸或者类型的图片，取回原始数据也很容易。

```java
// 指定DiskCacheStrategy
Glide.with(fragment)
  .load(url)
  .diskCacheStrategy(DiskCacheStrategy.ALL)
  .into(imageView)
```

##### 仅从缓存加载图片

某些情形下，你可能希望只要图片不在缓存中则加载直接失败（比如省流量模式）。如果要完成这个目标，你可以在单个请求的基础上使用onlyRetrieveFromCache方法。

````java
Glide.width(fragment)
  .load(url)
  .onlyRetrieveFromCache(true)
  .into(imageView)
````

##### 跳过缓存

如果你想确保一个特定的请求跳过磁盘或内存缓存（比如，图片验证码），Glide也提供了一些替代方案

```java
// 仅跳过内存缓存
Glide.width(fragment)
  .load(url)
  .skipMemoryCache(true)
  .into(View)
  
// 仅跳过磁盘缓存
Glide.width(fragment)
  .load(url)
  .diskCacheStrategy(DiskCacheStrategy.NONE)
  .into(view)
  
// 上面的策略也可以同时使用。虽然提供了这些办法让你跳过缓存，但你通常应该不会想这么做。从缓存中加载一个图片，要比拉取-解码-转换成一张新图片的完整流程快得多。 
```

#### 缓存的刷新

因为磁盘缓存使用的是哈希键，所以并没有一个比较好的方式来简单地删除某个特定url或文件路径对应的缓存文件。如果你只允许加载或缓存原始图片的话，问题可能会变得更简单，但因为Glide还会缓存缩略图和通过多种变换，它们中的任何一个都会导致缓存中创建一个新的文件，而要跟踪和删除一个图片都素有版本无疑是困难的。

在实践中，是缓存文件无效的最佳方式是在内容发生变化时（url, urI,文件路径等）更改你的标识符。

##### 定制缓存刷新策略

因为通常改变标识符比较困难或者根本不可能，所有Glide也提供了签名 API来混合额外数据到你的缓存键中。签名适用于媒体内容也适用于可以自行维护的一些版本元数据（注意hashcode方法的正确使用）。

请记住，为了比秒降低性能，您需要在后台批量加载任何版本元数据，以便在需要加载图像时即已处于可用状态。

如果这些努力都无法奏效，还不能更改标识符，也不能跟踪任何合理的版本元数据的情况下，也可以使用di sCacheStrategy（）和DiskCacheStrategy.NONE 来完全禁用磁盘缓存。



#### 资源管理

Glide的磁盘和内存缓存都是LRU，这意味着在达到使用限制或持续接近限定值之前，它们将占用持续增加的内存或磁盘空间。为了增加额外的灵活性Glide提供了一些额外的方式来让你可以管理你的应用使用资源。

请记住，更大的内存缓存、为徒池和磁盘缓存通常能提供更好的性能，或者至少在同等级别。如果你改变了缓存大小，你应该小心地测量一下你该懂之前和之后的性能对比，以确保你的修改带来的性价比是可以接受的。

##### 内存缓存

默认情况下Glide的内存缓存和BitmapPool会响应ComponentCallback2,并根据Android Fragmentwork提供的级别自动清理内容。因此你通常不需要尝试动态监视或清理你的缓存或BitmapPool。然而，如果必要的话，Glide确实提供了几个手动选项

- 永久尺寸调整 - 改变应用中Glide的可用RAM大小
- 暂时尺寸调整 - 在应用的特定部分暂时允许Glide使用更多或更少的内存，可以使用setmemoryVategory。
- 清理磁盘缓存 - 尝试清理素有的磁盘缓存条目，可以使用clearDiskCache。





#### 带问题阅读源码

1. 生命周期问题，Activity或者Fragment实例销毁时，Glide怎样自动取消并加载回收资源
2. ListView和RecyclerView中加载图片，Glide是怎样处理View的服用和请求的取消的
3. 