# Chapter 12: Layer Keyframe Animations & Struct Properties

图层上的关键帧动画与UIView上的关键帧动画略有不同。查看关键帧动画是将独立简单动画组合在一起的简单方法;它们可以为不同的视图和属性设置动画，动画可以重叠或在两者之间存在间隙。

相比之下，CAKeyframeAnimation允许您为给定图层上的单个属性设置动画。您可以定义动画的不同关键点，但动画中不能有任何间隙或重叠。尽管起初听起来有限制，但您可以使用CAKeyframeAnimation创建一些非常引人注目的效果。

在本章中，您将创建许多图层关键帧动画，从模拟真实世界碰撞的非常基本的动画到更高级的动画。在第15章“笔画和路径动画”中，您将学习如何进一步采用图层动画，并沿给定路径为图层设置动画。

现在，您将在跑步前走路，为您的第一层关键帧动画创建一个时髦的摇摆效果。

## 引入关键帧动画
想一想基本动画是如何工作的。 使用**fromValue**和**toValue**，**Core Animation**会在指定的持续时间内逐步修改这些值之间的特定图层属性。

例如，当您在45°和-45°（或π/ 4和-π/ 4之间）为您的数学类型旋转图层时，您只需要指定这两个值，并且图层渲染所有中间值以完成 动画：



[图片]



CAKeyframeAnimation使用一组值来动画命名值，而不是fromValue和toValue。 值的元素是动画的测量里程碑。 您还需要提供动画应达到每个值的关键点的时间。

看一下下面的简单图层关键帧动画示例：



[图片]



在上面的动画中，图层从45°旋转到-45°，但这次它有两个独立的阶段：首先，它在动画持续时间的前三分之二内从45°旋转到22°，然后旋转 在剩余的时间内一直到-45°。

实质上，使用关键帧设置动画层要求您为要设置动画的属性提供关键值，以及在0.0和1.0之间进行相应数量的相对关键时间。

## 创建图层关键帧动画
打开本章的入门项目，或者如果您完成了上一章中的项目，则可以从上次停止的地方开始。

打开**ViewController.swift**并找到`resetForm()`; 当您完成从“正在连接”到“授权”等不同登录状态消息的动画时，将执行此方法。 您将在虚假失败的身份验证尝试上略微为表单标题设置动画，以告知用户发生了错误。

将以下代码添加到`resetForm()`的末尾：

```swift
let wobble = CAKeyframeAnimation(keyPath: "transform.rotation")
wobble.duration = 0.25
wobble.repeatCount = 4
wobble.values = [0.0, -.pi/4.0, 0.0, .pi/4.0, 0.0]
wobble.keyTimes = [0.0, 0.25, 0.5, 0.75, 1.0]
heading.layer.add(wobble, forKey: nil)
```

在这里，您可以像通常对CABasicAnimation一样创建新的CAKeyframeAnimation：指定keyPath，设置动画的总持续时间并指示您希望它重复的次数。

然后在values数组中设置动画值：将图层从0°旋转到-45°（等于π/ 4），回到0°，一直旋转到45°，最后回到0°。 动画以相同的值开始和结束，这使得重复它成为一项简单的任务。

最后，设置动画值的关键时间，确保分别将开始和结束时间设置为0.0和1.0，以避免动画中的任何跳跃。

构建并运行您的项目; 点击登录按钮，等到序列结束，你会看到标题摆动，提醒用户他们的登录尝试失败：



[图片]



敏锐的读者可能已经注意到我还没有介绍结构属性的动画。 大多数情况下，你可以放弃动画结构的单个组件，例如CGPoint的x组件，或CATransformation3D的旋转组件，但是接下来你会发现动态结构值的动画比 你可能会先考虑一下。

## 动画结构值
结构实例是Swift中的一等公民。 事实上，在使用类和结构之间，语法上的差别很小。

但是，Core Animation是一个基于C构建的Objective-C框架，这意味着结构的处理方式截然不同。 Objective-C API喜欢处理对象，因此结构需要一些特殊的处理。

这就是为什么对图层属性（如颜色或数字）进行动画制作相对容易的原因，但为CGPoint等结构属性设置动画并不容易。

CALayer有许多可动画的属性，它们包含结构值，包括CGPoint类型的位置，CATransform3D类型的转换和CGRect类型的边界。 为了帮助解决这个问题，Cocoa包含了NSValue类，它将结构值“装入”或“包装”为对象。 NSValue附带了许多便利初始化程序，您可以将它们用于需要打包的每个结构，包括以下内容：

```swift
init(cgPoint: CGPoint)
init(cgSize: CGSize)
init(cgRect rect: CGRect)
init(caTransform3D: CATransform3D)
```

您将如何使用这些初始化程序来设置您的值？ 以下是使用CGPoint的示例位置动画：

```swift
let move = CABasicAnimation(keyPath: "position")
move.duration = 1.0
move.fromValue = NSValue(cgPoint: CGPoint(x: 100.0, y: 100.0))
move.toValue = NSValue(cgPoint: CGPoint(x: 200.0, y: 200.0))
```

如果您尝试将CGPoint直接分配给fromValue或toValue，则您的动画将无法按预期工作。 相反，您可以在将CGPoint分配给fromValue和toValue之前将其置于NSValue中。

关键帧动画也会发生同样的情况：如果您尝试将CGPoint实例数组指定为动画的值，则动画将无效，因为您必须使用盒装CGPoint NSValues数组。

本章的最后一部分将引导您完成装箱结构属性的装箱，同时添加最后一层关键帧动画，其中包括一个热气球！

## 中级关键帧动画

首先，您需要在屏幕上添加气球图像。 打开ViewController.swift，然后将以下代码添加到login（）的底部：

```swift
let balloon = CALayer()
balloon.contents = UIImage(named: "balloon")!.cgImage
balloon.frame = CGRect(x: -50.0, y: 0.0, width:
50.0, height: 65.0)
view.layer.insertSublayer(balloon, below: username.layer)
```

在上面的代码中，您将创建一个以气球图像作为其内容的新图层。

如果您需要在屏幕上显示图像但不需要使用UIView的所有好处（例如自动布局约束，附加手势识别器等），您可以简单地使用上面的代码示例中的CALayer。

您将图层定位在左上角附近，就在屏幕的可见区域之外。 最后，在“用户名”字段下方插入图层，以便气球显示在表单中的所有其他元素后面。

现在，您可以通过几个熟悉的步骤创建动画。 在上一代码下面添加以下代码：

```swift
let flight = CAKeyframeAnimation(keyPath: "position")
flight.duration = 12.0
```

在这里，您可以创建一个关键帧动画，并将其持续时间设置为12.0，这是人工身份验证过程的大致持续时间。

接下来，添加以下键值点和时间：

```swift
flight.values = [
  CGPoint(x: -50.0, y: 0.0),
  CGPoint(x: view.frame.width + 50.0, y: 160.0),
  CGPoint(x: -50.0, y: loginButton.center.y)
].map { NSValue(cgPoint: $0) }
flight.keyTimes = [0.0, 0.5, 1.0]
```

请注意如何使用map将点数组巧妙地转换为装在NSValues中的点数组。 斯威夫特不是很好吗？

这会沿着连接您指定给值的三个点的路径为气球设置动画，如下所示：



[图片]



添加最后几行以运行动画并设置气球图层的最终位置：

```swift
balloon.add(flight, forKey: nil)
balloon.position = CGPoint(x: -50.0, y: loginButton.center.y)
```

构建并运行您的项目; 只要点击“登录”按钮，就会看到气球飞过屏幕：



[图片]



您可以使用相同的技术为其他结构属性（如边界，位置和变换）设置动画。

如果要在更复杂的点集（例如平滑曲线路径）上为气球设置动画，请继续关注第15章“笔画和路径动画”，在这里您将学习如何使用以下任意路径为图层设置动画。 关键帧动画的特例。 您已经涵盖了本书中涉及基本图层动画的所有章节; 接下来的章节将向您介绍一些您可以使用专门的图层创建的很酷的动画。

## 关键点
* 您可以使用CAKeyframeAnimation类轻松创建图层关键帧动画。
* 与视图不同，图层关键帧动画通过几个可能的关键点为连续动画中的单个属性设置动画。
* 您可以通过将复杂属性数据类型包装为NSValue类型来为其设置动画。