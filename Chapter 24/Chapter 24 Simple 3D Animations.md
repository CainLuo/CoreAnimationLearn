# Chapter 24: Simple 3D Animations

在本章中，您将尝试新发现的有关相机距离和视角的知识。

设置图层的透视图后，您可以像往常一样处理图层的变换; 但现在您可以三维旋转，平移和缩放图层。

本章的项目具有折叠拉出菜单，在许多应用程序中普及，如Taasky：



[图片]



Office Buddy是一个办公室帮助应用程序，供员工访问有关日常公司生活的分类信息。

初始项目已经具有使菜单起作用的所有代码，但它仅适用于2D。 您的任务是将菜单带入第三维度并赋予其生命！

## 创建3D转换
打开本章的入门项目并构建并运行它以查看Office Buddy应用程序的初始版本：



[图片]



点击菜单按钮以显示侧面菜单; 或者，您可以向右滑动以显示侧面菜单。

正如您所看到的，这款应用程序与它们一样平坦。 但是，凭借您对3D视角的新见解，您将为菜单添加一些深度。

打开ContainerViewController.swift; 该控制器在屏幕上显示菜单视图控制器和内容视图控制器。 它还处理平移手势，以便用户可以打开和关闭菜单。

您的第一个任务是构建一个类方法，该方法为侧面菜单的给定百分比“开放性”创建相应的3D变换。

将以下方法声明添加到ContainerViewController.swift：

```swift
func menuTransform(percent: CGFloat) -> CATransform3D {
}
```

上面的方法接受菜单当前进度的单个参数，该参数由handleGesture（_ :)中的代码计算，并返回CATransform3D的实例。 您将直接将此方法的结果分配给菜单图层的transform属性。

将以下代码添加到新方法：

```swift
var identity = CATransform3DIdentity
identity.m34 = -1.0/1000
```

这段代码看起来有点令人惊讶; 到目前为止，您只使用函数来创建或修改变换。 但是，这一次，您正在修改其中一个类的属性。

> 注意：为什么这个属性叫做m34？ 视图和图层变换表示为二维数学矩阵。 对于图层变换矩阵，第4列第3行中的元素设置z轴透视图。 您可以直接设置此元素以应用所需的透视变换。

因此，您创建一个新的CATransform3D结构，并将其m34属性设置为-1.0 / 1000 ...为什么要选择该值？

好的，我会稍微备份一下。 要在图层上启用3D变换，您需要将m34设置为-1.0 / [摄像机距离]。 由于您仔细阅读了本节的介绍（您确实阅读过它，对吗？），您可以了解相机距离如何影响场景透视。

但为什么你使用1000作为相机距离？ 距离以相机和场景前方之间的点表示。 至于使用什么价值，事实是你需要尝试不同的值，看看什么看起来对你的特定动画有好处。

### 使用相机距离
对于普通应用程序中的UI元素，您可以参考以下参考资料，了解有关适当摄像机距离的一些指导：

* 0.1 ... 500：非常接近，有很多透视失真。
* 750 ... 2,000：视角不错，内容清晰可见。 
* 2,000及以上：几乎没有透视失真。

对于Office Buddy应用程序，1000点的距离将为菜单提供一个非常微妙的视角。 完成当前方法后，您可以看到它的运行情况。

将以下代码添加到menuTransform（percent :)的底部：

```swift
let remainingPercent = 1.0 - percent
let angle = remainingPercent * .pi * -0.5
```

在上面的代码中，您可以根据菜单的“开放性”值计算菜单的当前角度。

现在将以下代码添加到menuTransform（percent :)的底部：

```swift
let rotationTransform = CATransform3DRotate(
  identity, angle, 0.0, 1.0, 0.0)
let translationTransform = CATransform3DMakeTranslation(
    menuWidth * percent, 0, 0)
return CATransform3DConcat(rotationTransform, translationTransform)
```

在这里，您使用rotationTransform将图层绕y轴旋转。

菜单从左侧移动，因此您还可以创建平移变换以沿x轴移动它，最终将菜单宽度设置为100％。

最后，连接两个转换并返回结果。

现在，您可以使用menuTransform（percent :)来更新菜单转换，因为用户可以向右或向左平移。

从setMenu（toPercent :)中删除修改菜单原点的以下行：

```swift
menuViewController.view.frame.origin.x = menuWidth * CGFloat(percent) - menuWidth
```

您不再需要上面的行，因为您将通过其变换移动菜单。 将以下代码添加到setMenu（toPercent :)：

```swift
menuViewController.view.layer.transform = menuTransform(percent: percent)
```

构建并运行您的项目; 向右平移以查看菜单如何围绕其y轴旋转：



[图片]



菜单以3D形式旋转，但它围绕其水平中心旋转，从而将菜单与内容视图控制器分开。

### 移动图层的锚点
默认情况下，图层的锚点的x坐标为0.5，表示它位于中心。 您需要将锚点的x设置为1.0，以使菜单围绕其右边缘像铰链一样旋转，如下所示：



[图片]



所有变换都围绕图层的锚点计算。 设置图层的位置时，还可以使用锚点作为参考点。 这意味着在屏幕上设置其位置之前最好更改图层的anchorPoint。

在viewDidLoad（）中找到以下行，您可以在其中设置菜单框：

```swift
menuViewController.view.frame = CGRect(
  x: -menuWidth, y: 0,
  width: menuWidth, height: view.frame.height)
```

现在在该行上方插入以下代码（在设置视图帧之前插入行很重要，因为否则设置锚点将偏移视图 - 如果您想要看到差异，请尝试两种方式）：

```swift
menuViewController.view.layer.anchorPoint.x = 1.0
```

这会使菜单围绕其右边缘旋转。

再次构建并运行项目，然后在视图中水平平移并观察效果如何变化：



[图片]



那看起来好多了！

你差不多完成了 - 只需要处理几个比特。

### 通过着色创建透视图
阴影为3D动画带来了很多真实感; 为此，您将从内容视图控制器左侧的“阴影”中旋转菜单。

你没有在这里使用任何先进的着色技术; 相反，您可以通过在旋转时更改菜单的alpha来模拟这一点。

将以下代码添加到setMenu（toPercent :)：

```swift
menuViewController.view.alpha = CGFloat(max(0.2, percent))
```

在上面的代码中，您将百分比值直接指定给图层的alpha，但是将其限制为0.2以确保菜单在用户边缘时仍然可见，并且不会完全消失。

由于此应用程序的背景为黑色，因此降低菜单视图的alpha值会使菜单中显示黑色并模拟阴影效果。

构建并运行项目并观察效果：



[图片]



这是一个小细节，但它确实让你的3D动画“流行”！

您可能已经注意到，当您第一次点击菜单按钮时，动画不会以3D模式播放。 效果只会影响第二个及后续动画。

那是因为在第一次切换菜单之前，您没有设置3D动画参数和图层变换。 这很容易解决：菜单视图控制器加载后，将菜单进度设置为0.0以设置正确的菜单图层转换。

将以下内容添加到viewDidLoad（）的底部：

```swift
setMenu(toPercent: 0.0)
```

这将从一开始就使起始帧和层变换到位。 再次构建和运行您的项目; 点击菜单按钮，您将看到动画第一次正常工作。

### 光栅化效率
最后一项任务是让你的动画“完美”。 如果你在来回平移的同时盯着菜单，你会注意到菜单项的边框看起来像素化了。



[图片]



核心动画不断重绘菜单视图控制器的所有内容，并在所有元素移动时重新计算所有元素的透视失真，这不是非常有效 - 因此是锯齿状边缘。

最好让Core Animation知道您不会在动画期间更改菜单内容，以便它可以渲染菜单一次并简单地旋转渲染和缓存的图像。

这听起来很复杂，但你会发现它很容易实现。 滚动到handleGesture（）并找到.began案例块; 此代码在用户启动平移操作时执行。 您可以在此指示Core Animation缓存菜单。

将以下代码添加到.began案例块的末尾：

```swift
// Improve the look of the opening menu
menuViewController.view.layer.shouldRasterize = true
menuViewController.view.layer.rasterizationScale =
  UIScreen.main.scale
```

shouldRasterize指示Core Animation将图层内容缓存为图像。 然后设置rasterizationScale以匹配当前的屏幕比例，你就是金色的！

再次构建并运行项目，以查看图形的改进方式：



[图片]



核心动画在这个例子中真的很棒。 使用2D图像是这个框架真正做得很好的事情之一！

为避免在使用应用程序时进行任何不必要的缓存，您应该在动画完成后立即关闭光栅化。

在.failed案例中找到空动画完成闭包并添加以下代码：

```swift
self.menuViewController.view.layer.shouldRasterize = false
```

现在，您只在动画期间激活光栅化。 你效率如何！

只需几页，您就可以了解相机距离，透视以及如何设置3D场景并对其应用动画。

本节还有一章，其中包含更多3D乐趣 - 但在您开始之前，先看一下本章的挑战。

## 关键点
* CATransform3D的.m34值非常重要，因为它为您的3D变换提供了透视。
* 图层的锚点设置转换发生的点。

## 挑战
### 挑战1：创建自己的3D动画
对于此挑战，您将为菜单按钮创建3D旋转动画。 当用户平移按钮时，按钮将在菜单视图控制器旁边旋转。

具体来说，您将围绕x轴和y轴创建旋转，以使菜单按钮在其对角线上翻转。



[图片]



将以下代码添加到ContainerViewController.swift中的setMenu（toPercent :)：

```swift
let centerVC = centerViewController.viewControllers.first as?
CenterViewController
```

这将获取当前内容视图控制器，以便您可以使用它。

菜单按钮可通过CenterViewController的menuButton属性访问。

对于此挑战，请调整按钮的imageView的3D变换。如果您直接旋转按钮而不是使用3D变换，它将与下方的导航栏视图发生冲突，并可能被导航栏部分遮挡。

相比之下，按钮自己的2D空间在旋转之前已经位于所有导航栏视图的顶部 - 因此，如果旋转menuButton.imageView，它会在按钮自己的平面内旋转，该平面始终位于导航栏的顶部。

另一个问题是，第一次运行menuButton的代码将是nil，所以你应该将它视为一个可选的，即使它是隐式解包的。

对于旋转，请像在本章前面一样创建变换，但这次围绕x轴和y轴旋转视图。如有必要，请查看CATransform3DRotate（）的文档。

这次你不需要计算剩余的百分比; 要获得旋转角度，只需将进度乘以.pi即可。

当您完成解决方案时，按钮图像应在您平移屏幕时翻转，如下所示：



[图片]



而已！ 进入下一章，了解3D中更棒的动画！

