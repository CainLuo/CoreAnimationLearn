# Chapter 14: Gradient Animations

iOS的许多外观和感觉来自UI中非常微妙的动画。

虽然它不再是iOS的一部分，但其中最好的是一个简单的小动画：锁定屏幕上的“滑动解锁”标签。

在本章中，您将学习如何使用移动渐变模拟此效果以及如何为这些渐变的颜色和布局设置动画：



[图片]



您将为“幻灯片显示”标签设置渐变动画，然后在用户滑过标签时显示一个很酷的神秘效果。 但是，你必须完成本章，看看这个很酷的效果是什么！

作为额外的奖励，您将学习如何从一段文本中创建图层蒙版并使用它来掩盖渐变。

## 绘制第一个渐变
打开本章的入门项目，然后选择Main.storyboard以查看UI当前的外观：



[图片]



顶部有一个静态标签，模仿锁定屏幕上的iPhone时钟和底部附近的另一个视图。

底视图是AnimatedMaskLabel的一个实例，它包含在starter项目中。 在本章中，您将使用此类添加渐变动画。

构建并运行您的项目; 你会看到屏幕顶部只显示虚假时钟：



[图片]



您将首先绘制AnimatedMaskLabel的基本渐变。 在下面显示的注释之后，将以下代码添加到gradientLayer属性代码中的AnimatedMaskLabel.swift：

```swift
// Configure the gradient here
gradientLayer.startPoint = CGPoint(x: 0.0, y: 0.5)
gradientLayer.endPoint = CGPoint(x: 1.0, y: 0.5)
```

这定义了渐变的方向及其起点和终点。



[图片]



现在添加以下代码以定义在刚刚添加的代码之后构建渐变的颜色：

```swift
let colors = [
  UIColor.black.cgColor,
  UIColor.white.cgColor,
  UIColor.black.cgColor
]
gradientLayer.colors = colors
```

上面的渐变以黑色开始，混合为白色，最后混合为黑色。

您还可以指定这些颜色应该出现在渐变框架中的确切位置。 在下面添加以下代码：

```swift
let locations: [NSNumber] = [
  0.25,
	0.5,
	0.75
]
gradientLayer.locations = locations
```

这将设置渐变颜色里程碑，如下所示：



[图片]



您可以拥有任意数量的关键点和颜色里程碑，但本章中的文本渐变动画只需要上面显示的简单的黑白黑色渐变。

将以下代码添加到layoutSubviews（）以为渐变添加框架：

```swift
gradientLayer.frame = bounds
```

您现在需要做的就是将渐变添加到视图的图层以查看其实际效果。 将下面的代码行添加到didMoveToWindow（）的末尾：

```swift
layer.addSublayer(gradientLayer)
```

构建并运行您的项目; 你应该看到应用程序显示你正在寻找的确切渐变：



[图片]



这是一个很好的开始！ 现在您需要弄清楚如何为此渐变设置动画。

## 动画渐变
CAGradientLayer为您提供了四个可动画的属性以及从CALayer继承的属性：

* colors：为渐变的颜色设置动画，使其具有色调。
* locations：为颜色里程碑位置设置动画以使颜色四处移动
  在渐变内。
* startPoint和endPoint：为渐变布局的范围设置动画。 

在本节中，您将为位置设置动画以使渐变“移动”。

将以下代码添加到didMoveToWindow（）的末尾：

```swift
let gradientAnimation = CABasicAnimation(keyPath: "locations")
gradientAnimation.fromValue = [0.0, 0.0, 0.25]
gradientAnimation.toValue = [0.75, 1.0, 1.0]
gradientAnimation.duration = 3.0
gradientAnimation.repeatCount = Float.infinity
```

在此图层动画中，首先将三个颜色里程碑推到渐变框架的左边缘，然后将所有三个里程碑推向右边缘结束动画：



[图片]



由于您将repeatCount设置为无穷大，动画将持续3秒并将永远重复。

最后，将以下行添加到didMoveToWindow（）的末尾：

```swift
gradientLayer.add(gradientAnimation, forKey: nil)
```

这会将动画添加到渐变图层。 构建并运行您的项目，您将看到动画形成：



[图片]



这看起来很不错，但渐变非常刺眼，特别是在中间附近。 没问题：只需放大渐变边界，你就会得到更温和的渐变。

在`layoutSubviews()`中找到`gradientLayer.frame = bounds`行，并将其替换为以下为渐变图层设置更大框架的代码：

```swift
gradientLayer.frame = CGRect(
  x: -bounds.size.width,
  y: bounds.origin.y,
  width: 3 * bounds.size.width,
  height: bounds.size.height)
```

这会将渐变框设置为可见区域宽度的三倍。 动画进入视图，直接通过它，并从右侧退出：



[图片]



构建并运行项目以查看更改的外观：



[图片]



这看起来更像是你想要的平滑渐变。 现在您已经拥有渐变，您需要创建文本图层以用作蒙版。

## 创建文本掩码
在本节中，您将渲染存储在AnimatedMaskLabel的text属性中的字符串，并使用它来屏蔽渐变图层。 在AnimatedMaskLabel类中创建一个新的常量属性，以保存文本属性，如下所示：

```swift
let textAttributes: [NSAttributedString.Key: Any] = {
  let style = NSMutableParagraphStyle()
  style.alignment = .center
  return [
    .font: UIFont(
      name: "HelveticaNeue-Thin",
      size: 28.0)!,
    .paragraphStyle: style
  ]
}()
```

接下来，您需要将文本渲染为图像。 执行此操作的自然位置是text属性的属性观察者。 在setNeedsDisplay（）调用之后添加以下代码：

```swift
let image = UIGraphicsImageRenderer(size: bounds.size)
  .image { _ in
    text.draw(in: bounds, withAttributes: textAttributes)
}
```

在这里，您可以使用图像渲染器来设置上下文，绘制到它，并将结果作为UIImage获取。 现在，您可以使用该图像在渐变图层上创建蒙版。 为此，首先从图像中创建一个图层，如下所示：

```swift
let maskLayer = CALayer()
maskLayer.backgroundColor = UIColor.clear.cgColor
maskLayer.frame = bounds.offsetBy(dx: bounds.size.width, dy: 0)
maskLayer.contents = image.cgImage
```

只需使用CALayer的默认初始值设定项，即可将maskLayer创建为空图层。 然后，您将设置一个完全透明的图层背景，因为您将使用该图层作为蒙版。 然后用图层的宽度偏移图层框架; 这样，蒙版将显示在渐变的中心。 这是必要的，因为“拉伸”渐变的宽度目前是可见视图的三倍。 最后，将图像对象直接分配给图层的contents属性。

再添加一行以将新图层设置为渐变的蒙版：

```swift
gradientLayer.mask = maskLayer
```

构建并运行您的项目以查看完整开发的动画：



[图片]



嘿 - 看起来很光滑！ 但是，当用户在标签上滑动时，您还没有发现显示的内容 - 您是否仅限于渐变的单色调色板？ 所有这些都将被揭示 - 当您完成下面的挑战时！

## 关键点
* 您可以使用CAGradientLayer并设置渐变颜色在屏幕上绘制渐变。
* 您可以通过在CAGradientLayer上设置颜色，startPoint和endPoint属性的动画来创建渐变动画。
* 您可以设置渐变以通过多个颜色关键点（并为其设置动画）来改变其色调，以创建更多迷幻的视觉效果。

## 挑战
我知道悬念正在杀死你; 这两个挑战将为标签添加一个幻灯片手势识别器，并为渐变动画添加一个额外的颜色效果。
### 挑战1：滑动以显示手势识别器
打开ViewController.swift并将以下代码添加到viewDidLoad（）：

```swift
let swipe = UISwipeGestureRecognizer(target: self,
  action: #selector(ViewController.didSlide))
swipe.direction = .right
slideView.addGestureRecognizer(swipe)
```

这将创建一个从右到右的手势识别器并将其附加到slideView。 识别器将在ViewController上调用didSlide（）。

didSlide（）已经为您实现了，所以您现在需要做的就是启动应用程序并将手指滑过动画标签以显示下方的内容。

### 挑战2：迷幻渐变动画
在本章的最后一个挑战中，您将尝试为渐变添加更多颜色并观察效果。

如果您想要指导您的探索，请尝试使用以下渐变颜色列表：

```swift
UIColor.yellow
UIColor.green
UIColor.orange
UIColor.cyan
UIColor.red
UIColor.yellow
```

您还需要调整位置值以保持所有颜色的顺序。 使用以下位置开始动画：0.0,0.0,0.0,0.0,0.0和0.25。 然后将位置设置为：0.65,0.8,0.85,0.9,0.95和1.0。

构建并运行您的项目; 玩弄动画和各种参数，直到你把它调整到你自己的设计偏好：



[图片]



这使章节结束; 你已经看到动画渐变有多容易，以及如何用文本图层实现一些高级图层蒙版技巧。

下一章将介绍形状的笔画动画，这是本书关于图层动画的这一部分中的最后一个主题; 您将学习如何以交互方式绘制形状，作为奖励，您将学习高级关键帧动画。