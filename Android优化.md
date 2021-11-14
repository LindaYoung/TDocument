### 一、资源混淆7zip压缩

- 增加破解难度，用无意义的字母替换掉资源文件目录及名称。如res/drawable/wechat变为r/d/a。
- 通过缩短混淆资源ID长度的同时利用7z深度压缩，减少打包体积   云电脑：46M到35M 减少24%左右

#### 原理

通过resource.arsc文件格式，混淆步骤为：

1. 修改arsc文件，主要为全局与资源名字符串池
2. 修改字符串池中的字符串，以无意义的a b替换
3. 需改apk中的res目录字眼文件名
4. 打包（7zip）、对齐、签名

目前的可实施方案有微信资源混淆打包工具AndResGuard和美团方案



[Android应用程序资源的编译和打包过程分析](https://blog.csdn.net/luoshengyang/article/details/8744683)

[微信资源混淆原理](https://blog.csdn.net/cg_wang/article/details/70183864)

### 二、网络优化

##### 网络基础：

单工：信息只能由一方A传到另一方B。无线广播

半双工：信息在同一时刻只能从一方传到另一方。http

全双工：线路上存在A到B和B到A的双向信号传输。websocket（应用层协议）、socket（接口协议）、rtsp

http connection：keep—Alive 1.0默认fasle。1.1之后默认true。表示是长链接。



##### 兼容性

服务器返回数据类型有可能跟约定的不一样，客户端可以做一个兼容。

Gson 的GsonBuilder可以registerTypeHierarchyAdapter()里进行兼容转化

```java
private static final Gson GSON = new GsonBuilder()
			.registerTypeAdapter(Integer.class, new IntegerDefaultAdapter())
			.registerTypeAdapter(int.class, new IntegerDefaultAdapter())
			.registerTypeAdapter(Long.class, new LongDefaultAdapter())
			.registerTypeAdapter(long.class, new LongDefaultAdapter())
			.registerTypeAdapter(Double.class, new DoubleDefaultAdapter())
			.registerTypeAdapter(double.class, new DoubleDefaultAdapter())
			.registerTypeAdapter(Float.class, new FloatDefaultAdapter())
			.registerTypeAdapter(float.class, new FloatDefaultAdapter())
			.serializeNulls() // 默认情况下序列化的时候是不导出值是null的属性的,配之下会导出，方便调试。
			.create();


public class IntegerDefaultAdapter implements JsonSerializer<Integer>, JsonDeserializer<Integer> {
    @Override
    public Integer deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) throws JsonParseException {
        if (TextUtils.isEmpty(json.getAsString())) return -1;

        return json.getAsInt();
    }

    @Override
    public JsonElement serialize(Integer src, Type typeOfSrc, JsonSerializationContext context) {
        return new JsonPrimitive(src);
    }
}
```

##### 可靠性

##### 安全性

http和https的区别，多了个SSL

SSL 加密解密用的

对称性加密：加密和解密用的是同一个密钥。

非对称加密：加密和解密用的不是同一个密钥。

https通过非对称加密建立连接，后面传递的时候用对称加密。

非对称加密怎么保证公钥的安全，怎么知道你拿的私钥就可以直接解密这段。去公证处，会得到数字签名。

- 公钥+个人信息通过Hash算法形成消息摘要
- 消息摘要通过CA的私钥加密形成数字签名
- 公钥+数字签名 形成一个数字证书
- 数字签名通过CA的公钥解密出来的信息摘要 和 公钥Hash算法后得到的消息摘要是否相同。如果相同则说明是同一个，可以进行解密。

##### 弱网优化

网络连不上：unknoehost 、dns 有可能污染，域名无法解析（需要备案，域名和空间都在国内）

okhttpClientBuilder.dns(new TestDns()) 来配置个自己保底的一个域名。TestDns实现Dns的lookup函数，先用系统找出来的，如果找不到就用我们自己的ip保底。



### 二、图片优化

#### webp格式

WebP是google提供的一种支持有损压缩和无损压缩的图片文件格式，而且可以提供比jpeg或png更好的压缩。AndroidStudio4.0（API level 14）中支持有损的webP图像，在Android4.3（API level 18）和更高版本中支持无损和透明的webP图像。

android studio中使用：右键图片 --> convert to webP --> (默认保留75的画质，肉眼看不出大区别) OK

png --> webp  2.82KB --> 0.987K 减少了65%

tiny 1.8K ->605 减少了60%-70%  : webp保留75%的的图片质量：1.8-->624b; 保存100%的画质518B



动图：

- 前段实现svga动画（实现成本高、维护成本高，容易有买家秀/买家秀区别、客户端不能服用）
- 设计师且gif（文件较大、容易有锯齿）
- png序列（文件较大、容易有锯齿）
- lottie，lottie是一个库，可以解析使用AE制作的动画（需要bodymovin导出为json格式），支持web、ios、android和rect native。

#### lottie

在web侧，lottie-web库可以解析道出的动画json文件，并将其以svg或者canvas的方法将动画绘制到我们页面中。

lottie方案，json文件大小会比gif文件小很多，性能也会更好。

缺点：

- lottie-web文件本身仍然比较大
- lottie动画其实可以理解为svg动画/canvas动画，不能给已存在的html添加动画效果
- 动画json文件的导出，目前是将AE里面的参数一一导出成json内容，如果设计师建了很多的图层，可能仍然有json文件比较大（20kb）的问题，需要设计师遵循一定的规范。
- 有很少量的AE动画效果，lottie无法实现，有些事因为性能问题，有些事没法做。比如：描边动画
- 

