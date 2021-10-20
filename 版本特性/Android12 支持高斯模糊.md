# Android12 支持高斯模糊

## 1. 高斯模糊：

<center class="half">
    <img src="https://cdn.jsdelivr.net/gh/kobeyytt1996/imgbed/yuanyuan/image-20211020182343740.png" height="300"/>
	 <img src="https://cdn.jsdelivr.net/gh/kobeyytt1996/imgbed/yuanyuan/image-20211020182235726.png" height="300"/>

高斯模糊是是图片产生模糊效果的一种算法，使用正态分布来平滑数据。

#### 1.  原理：

所谓"模糊"，可以理解成每一个像素都取周边像素的平均值。下图中，2是中间点，周边点都是1。"中间点"取"周围点"的平均值，就会变成1，在图形上，就相当于产生"模糊"效果，"中间点"失去细节。

<center class="half">
    <img src="https://cdn.jsdelivr.net/gh/kobeyytt1996/imgbed/yuanyuan/bg2012111403.png" height="300"/>
	 <img src="https://cdn.jsdelivr.net/gh/kobeyytt1996/imgbed/yuanyuan/bg2012111404.png" height="300"/>
</center>

显然，计算平均值时，取值范围越大，"模糊效果"越强烈。下面分别是原图、模糊半径3像素、模糊半径10像素的效果。模糊半径越大，图像就越模糊。

![img](https://cdn.jsdelivr.net/gh/kobeyytt1996/imgbed/yuanyuan/bg2012111406.jpg)

接下来的问题就是，既然每个点都要取周边像素的平均值，那么应该如何分配权重呢。如果使用简单平均，显然不是很合理，因为图像都是连续的，越靠近的点关系越密切，越远离的点关系越疏远。因此，加权平均更合理，距离越近的点权重越大，距离越远的点权重越小。

#### 2. 正态分布（高斯分布）的权重

下图分别是正态分布一维和二维的密度曲线：

<center class="half">
    <img src="https://cdn.jsdelivr.net/gh/kobeyytt1996/imgbed/yuanyuan/bg2012111407.png" height="300" width="500"/>
	 <img src="https://cdn.jsdelivr.net/gh/kobeyytt1996/imgbed/yuanyuan/bg2012110708.png" height="300"/>
</center>
正态分布的密度函数叫做"高斯函数"（Gaussian function）。它的一维形式是：

![img](https://cdn.jsdelivr.net/gh/kobeyytt1996/imgbed/yuanyuan/sigma%5E%7B2%7D%7D&chs=120.png)

二维高斯函数：

![img](https://cdn.jsdelivr.net/gh/kobeyytt1996/imgbed/yuanyuan/sigma%5E2%7D&chs=80.png)

#### 3. 权重矩阵

假定中心点的坐标是（0,0），那么距离它最近的8个点的坐标如下，更远的点以此类推：

![img](https://cdn.jsdelivr.net/gh/kobeyytt1996/imgbed/yuanyuan/bg2012111410.png)

为了计算权重矩阵，需要设定σ的值。假定σ=1.5，则模糊半径为1的权重矩阵如下：

![img](https://cdn.jsdelivr.net/gh/kobeyytt1996/imgbed/yuanyuan/bg2012111411.png)

这9个点的权重总和等于0.4787147，如果只计算这9个点的加权平均，还必须让它们的权重之和等于1，因此上面9个值还要分别除以0.4787147，得到最终的权重矩阵：

![img](https://cdn.jsdelivr.net/gh/kobeyytt1996/imgbed/yuanyuan/bg2012111412.png)

#### 4. 计算高斯模糊：

有了权重矩阵，就可以计算高斯模糊的值了。假设某黑白图片中现有9个像素点，灰度值（0-255）如下：

![img](https://cdn.jsdelivr.net/gh/kobeyytt1996/imgbed/yuanyuan/bg2012111413.png)

每个点乘以自己的权重值，得到：

![img](https://cdn.jsdelivr.net/gh/kobeyytt1996/imgbed/yuanyuan/bg2012111416.png)

将这9个值加起来，就是中心点的高斯模糊的值。对所有点重复这个过程，就得到了高斯模糊后的图像。如果原图是彩色图片，可以对RGB三个通道分别做高斯模糊。如果一个点处于图片边界，周边没有足够的点，那会有几种常用方式来处理，比如直接重复边界点。这些方法的视觉上区别不明显。

## 2. Android12 新增高斯模糊API：

Android 12 中，可以更轻松地将常用图形效果应用于View上，View中增加了setRenderEffect接口：

``` java
public void setRenderEffect(@Nullable RenderEffect renderEffect) {
        ...
}
```

 RenderEffect是新增的设置渲染特效的类，可将模糊、色彩滤镜等特效直接应用于View上。如果参数传null，则去除掉之前的特效。

RenderEffect中高斯模糊的对应静态方法：

``` java
public static RenderEffect createBlurEffect(float radiusX, float radiusY, @NonNull TileMode edgeTreatment)
```

- radiusX和radiusY指在X轴和Y轴上进行高斯模糊的半径
- edgeTreatment用于如何模糊图片边缘附近的内容，共有四种模式

**示例：**

``` kotlin
imageView.setRenderEffect(RenderEffect.createBlurEffect(3F, 3F, Shader.TileMode.CLAMP))
```

即对imageView的X轴和Y轴都取半径为3个像素点进行高斯模糊，且边界点采用重复的方式。效果如下：

createBlurEffect还有四个参数的版本：

``` java
public static RenderEffect createBlurEffect(float radiusX, float radiusY, 
              @NonNull RenderEffect inputEffect, @NonNull TileMode edgeTreatment)
```

- inputEffect指先做一次相应的特效渲染，再进行高斯模糊。

RenderEffect还提供了很多其他特效的对应方法：

![image-20211020165452014](https://cdn.jsdelivr.net/gh/kobeyytt1996/imgbed/yuanyuan/image-20211020165452014.png)

**需要注意，在低于Android 12的设备上使用会导致崩溃**。

## 3. 和以前方式的对比：

RenderScript是在Android上的高性能运行密集型运算的框架，以前实现高斯模糊通常是用这个库。调用方式如下：

``` kotlin
private fun getBlurBitmap(radius: Int, bitmap: Bitmap): Bitmap {
        val renderScript = RenderScript.create(this)
        val input: Allocation = Allocation.createFromBitmap(renderScript, bitmap)
        val output: Allocation = Allocation.createTyped(renderScript, input.type)
        val scriptIntrinsicBlur: ScriptIntrinsicBlur =
            ScriptIntrinsicBlur.create(renderScript, Element.U8_4(renderScript))
        scriptIntrinsicBlur.setRadius(radius.toFloat())
        scriptIntrinsicBlur.setInput(input)
        scriptIntrinsicBlur.forEach(output)
        output.copyTo(bitmap)
        return bitmap
    }
```

获取到模糊的图片，再set到对应的View中。很明显，Android12提供的API大大简化了这个过程且更加灵活。

虽然官方宣称渲染性能得到提升，但使用一张200k的图片测试，用两种方式运行后，分别查看了内存和耗时，基本一致，可能在较大图片上才存在差距。
