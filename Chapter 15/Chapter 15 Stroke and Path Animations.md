# Chapter 15: Stroke and Path Animations

本章包含了本书的图层动画部分; 您将了解中风和路径动画，因为您在现有的Pack List项目中添加了一个很酷的拉动 - 刷新动画，当应用程序假装从Internet获取新数据时，该项目可以为用户提供娱乐：



[图片]



在此过程中，您将学习如何为形状绘制设置动画，作为奖励，您将看到一种特殊的关键帧动画，您可以使用它来沿任意路径移动对象。

## 创建交互式笔画动画
打开本章的入门项目，然后构建并运行它以查看UI的外观：



[图片]



ViewController.swift中现有代码可以为您填充包含许多休假项目的表格。 拉下桌子，你会看到屏幕顶部出现一个刷新视图：



[图片]



刷新视图保持可见状态四秒钟，然后缩回。 你在这里的工作是添加一个有趣的动画来招待用户等待。

刷新视图已包含拉动和释放动作的所有代码; 你只需要担心添加动画。

> 注意：下拉刷新代码基于我们的一个视频教程。 如果您想了解更多有关它如何工作的信息，请访问以下链接查看Swift Scroll View School视频系列：https://videos.raywenderlich.com。

构建动画的第一步是创建一个圆形。 打开RefreshView.swift并将以下代码添加到`init(frame:scrollView:)`：

```swift
ovalShapeLayer.strokeColor = UIColor.white.cgColor
ovalShapeLayer.fillColor = UIColor.clear.cgColor
ovalShapeLayer.lineWidth = 4.0
ovalShapeLayer.lineDashPattern = [2, 3]

let refreshRadius = frame.size.height/2 * 0.8

ovalShapeLayer.path = UIBezierPath(ovalIn: CGRect(
  x: frame.size.width/2 - refreshRadius,
  y: frame.size.height/2 - refreshRadius,
  width: 2 * refreshRadius,
  height: 2 * refreshRadius)
).cgPath

layer.addSublayer(ovalShapeLayer)
```

ovalShapeLayer是类型为CAShapeLayer的RefreshView上的属性。 你已经非常熟悉形状层; 在这里，您只需设置笔触和填充颜色，并将圆直径设置为视图高度的80％，这样可确保在形状周围形成舒适的边距。

您还没有遇到过上面代码中的一个属性：lineDashPattern。 此属性允许您为形状笔划设置虚线模式; 您只需提供一个数组，其中包含短划线的长度和间隙的长度（以像素为单位）。

构建并运行项目以查看圆圈的外观：



[图片]



这看起来非常好 - 而且很容易创建。 这将为您提供圆形进度条。

在RefreshView中，只要用户通过scrollViewDidScroll（_ scrollView :)滚动，就会调用redrawFromProgress（）; 这使它成为更新进度条视觉效果的便利位置。

将以下代码添加到`redrawFromProgress()`：

```swift
ovalShapeLayer.strokeEnd = progress
```

随着进度从0.0增加到1.0，形状的行程末端向前移动到行程的起始点。

这是进度为0.25时形状的外观：



[图片]



这是它的中途，在0.5：



[图片]



构建并运行您的项目; 向上和向下拖动表格视图以查看笔划长度的变化。

现在，您将为刷新控件添加一个看起来很酷的飞机。

滚动回init（frame：scrollView :)并将以下代码添加到初始化程序的底部：

```swift
let airplaneImage = UIImage(named: "airplane.png")!
airplaneLayer.contents = airplaneImage.cgImage
airplaneLayer.bounds = CGRect(x: 0.0, y: 0.0,
  width: airplaneImage.size.width,
  height: airplaneImage.size.height)
airplaneLayer.position = CGPoint(
  x: frame.size.width/2 + frame.size.height/2 * 0.8,
  y: frame.size.height/2)
layer.addSublayer(airplaneLayer)
```

代码应该看起来很熟悉。 在前面的章节中，您已经完成了几次这样的操作。 您只需加载airplane.png并将其指定为屏幕上图层的内容，然后将飞机图层定位在圆开始绘制的位置。

构建并运行项目，以便在向下拉表格视图时显示飞机：



[图片]



当用户拉下桌子时，飞机应该淡入。 

将以下代码添加到init（frame：scrollView :)：

```swift
airplaneLayer.opacity = 0.0
```

这使得飞机开始时完全透明。

现在将以下代码添加到redrawFromProgress（）以在用户下拉时逐步更改飞机图层的不透明度：

```swift
airplaneLayer.opacity = Float(progress)
```

opacity的类型为Float，因此您需要从CGFloat转换进度。

构建并运行您的项目; 拉下桌子，你应该看到飞机逐渐出现：



[图片]



这包含了本章的交互式动画部分。 下一部分将引导您完成动画进度指示器，以便在用户等待虚假刷新数据时保持用户参与。

## 动画两个笔画结束
在本节中，您将为strokeStart和strokeEnd属性设置动画，以使形状“四处奔跑”。

当应用程序开始获取虚假数据时，您将启动动画。 将以下代码添加到beginRefreshing（）的末尾：

```swift
let strokeStartAnimation = CABasicAnimation(
  keyPath: "strokeStart")
strokeStartAnimation.fromValue = -0.5
strokeStartAnimation.toValue = 1.0

let strokeEndAnimation = CABasicAnimation(
  keyPath: "strokeEnd")
strokeEndAnimation.fromValue = 0.0
strokeEndAnimation.toValue = 1.0
```

此代码创建两个动画：第一个动画将strokeStart从-0.5设置为1.0。 这是一个简单而廉价的动画技巧; 虽然从-0.5到0.0的值动画没有任何反应，因为这些属性的所有负值都只意味着形状的任何部分都不可见。

这给了第二个动画 -  strokeEnd上的一个动画 - 有点先行。 这会在屏幕上绘制一小部分形状，直到strokeStart在动画结束时赶上strokeEnd。

在beginRefreshing（）的末尾添加以下代码以同时运行两个动画：

```swift
let strokeAnimationGroup = CAAnimationGroup()
strokeAnimationGroup.duration = 1.5
strokeAnimationGroup.repeatDuration = 5.0
strokeAnimationGroup.animations =
  [strokeStartAnimation, strokeEndAnimation]
ovalShapeLayer.add(strokeAnimationGroup, forKey: nil)
```

在上面的代码中，您创建一个动画组并重复动画五次。 这应该足够长，以便在刷新视图可见时保持动画运行。 然后，将两个动画添加到组中，并将组添加到进度条。

构建并运行您的项目; 拉动并释放表格以查看动画中的动画：



[图片]



您刚刚创建了自己的自定义微调器！ 虽然它看起来非常整洁，但只需稍加努力就可以让它变得更酷 - 并且可以从图层路径动画中获得一些帮助！

## 创建路径关键帧动画
您了解了如何使用关键帧动画和第12章“关键帧动画和结构属性”中的values属性为图层设置动画。 要沿着路径设置图层动画，您可以执行相同的操作，但您可以将CGPath指定给动画的路径属性。

然后，Core Animation将计算沿着CGPath的图层的中间位置，并在动画的持续时间内很好地设置动画。

将以下代码添加到beginRefreshing（）：

```swift
let flightAnimation = CAKeyframeAnimation(keyPath: "position")
flightAnimation.path = ovalShapeLayer.path
flightAnimation.calculationMode = .paced
```

在这里，您可以创建一个CAKeyframeAnimation，就像之前一样，将动画属性设置为position。 但是这次你为路径分配一个值。 在这种情况下，您可以重复使用ovalShapeLayer的圆形路径。

最后，将动画计算模式设置为节奏模式 - 这将确保图层沿路径平滑动画。

calculateMode是另一种控制动画时间的方法。当您将该属性设置为.paced时，Core Animation会以恒定的速度为您的图层设置动画，忽略您设置的任何关键时间。这对于在任意路径上生成平滑动画非常有用。

CAAnimationCalculationMode结构提供了更多常量。一个有趣的是.discrete。此计算模式使Core Animation从键值跳转到键值而不进行任何插值。是的，你做得对 - 核心动画有一个特殊的模式来制作动画，不动画任何东西。

足够的计算模式乐趣 - 回到手头的任务。

现在，您需要在此动画中创建一个组并在飞机图层上运行它。您稍后需要该组，因为您将添加第二个动画以补充第一个动画。

将以下代码添加到beginRefreshing（）：

```swift
let flightAnimationGroup = CAAnimationGroup()
flightAnimationGroup.duration = 1.5
flightAnimationGroup.repeatDuration = 5.0
flightAnimationGroup.animations = [flightAnimation]
airplaneLayer.add(flightAnimationGroup, forKey: nil)
```

在上面的代码中，您创建组并为其提供与进度条描边动画相同的持续时间和重复计数。 然后，您将flightAnimation添加为组中的唯一项目，并将该组添加到飞机图层。

运行动画并查看飞行员的技能：



[图片]



你从来没有见过像这样的航展：飞机在天空中执行循环并保持完全垂直！ 虽然这很有趣，但你应该让飞机更自然地飞翔。

> 注意：CAKeyframeAnimation有一个名为rotationMode的属性，类型为CAAnimationRotationMode，当设置为.rotateAuto时，它会自动将图层指向它正在移动的方向。 但是，您将在本章中手动创建此效果，因为对于简单的圆形路径来说，这是一项简单的任务。

在创建flightAnimationGroup的行上方插入以下新动画代码：

```swift
let airplaneOrientationAnimation = CABasicAnimation(keyPath:
"transform.rotation")
airplaneOrientationAnimation.fromValue = 0
airplaneOrientationAnimation.toValue = 2.0 * .pi
```

airplaneOrientationAnimation通过其变换将图层旋转整整360度 - 从0到2π。 换句话说，飞机图像将围绕其中心旋转一整圈，并且由于您同时沿圆形路径移动飞机层，因此飞行最终看起来很自然。

现在您只需要将airplaneOrientationAnimation添加到动画组。

将新动画添加到数组中，如下所示：

```swift
flightAnimationGroup.animations = [flightAnimation, airplaneOrientationAnimation]
```

构建并运行您的项目并享受完整的下拉到刷新控件：



[图片]



如果您想尝试空中杂技，可以尝试将旋转动画设置为0到4π - 非常有趣！


请注意，您不仅限于简单路径，例如方形或圆形。 您可以将任何CGPath提供给关键帧动画，并沿着一些极其复杂的路径发送图层。

如果你要创建更复杂的路径动画并希望保持理智，那么你应该记住本章前面关于rotationMode的注释。

## 关键点
* 您可以通过设置CAShapeLayer的strokeStart和strokeEnd属性的动画来设置在屏幕上绘制形状的过程的动画。
* 您可以使用关键帧动画在屏幕上沿路径设置图层动画，并使用给定的CGPath为图层的位置属性设置动画。

## 部分结论
这包含了基本层动画部分。 你已经经历了很多 - 并且沿途学到了很多东西！
在本书的这一部分中，您了解到：

* 基本移动，淡入淡出，旋转和缩放动画
* 组和关键帧动画
* 形状，蒙版和渐变动画
* 笔画和路径动画

下一章将指导您完成一个全新的专业领域 - 制作您自己的动画克隆！

