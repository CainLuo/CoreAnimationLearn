# Chapter 2: Springs

UIKit中的spring提供了更多的优美和美观来查看动画。苹果自己也使用spring来制作大量的系统动画以及应用程序。spring通常负责您在屏幕上看到的任何弹跳视图，但是在处理spring动画参数时，您将看到这个API允许您创建各种令人愉快的动画。

到目前为止，您的动画都是单向流动的。当你动画视图的位置时，它是从点a到点B的简单运动，就像这样:



![图片](https://raw.githubusercontent.com/CainLuo/CoreAnimationLearn/master/Chapter%202/1.png)



在本章中，您将学习如何创建更复杂的动画，它可以移动视图，就像它们被附加到一个spring上一样，如下所示:



![图片](https://raw.githubusercontent.com/CainLuo/CoreAnimationLearn/master/Chapter%202/2.png)



如果将基本动画从A点移动到B点，并添加一点弹性，动画的运动将遵循如下红色箭头所示的路径:



![图片](https://raw.githubusercontent.com/CainLuo/CoreAnimationLearn/master/Chapter%202/3.png)



视图从A点指向B点，但是稍微超过了B点。然后视图返回到点B，这次的超调稍微小一些。这种前后振荡不断重复，直到视图在点B处停止。

效果很好;它给你的动画增加了一种时髦的、真实的感觉。本章将向您展示如何使用这种效果来为UI添加一点趣味性。

## 弹簧的动画
你将继续上一章的项目;如果您没有完成第1章中的练习(包括本章末尾的挑战)，那么从第2章的Resources文件夹中获取starter项目，然后从那里开始。

建立和运行您的项目;当应用程序像这样打开时，你应该会看到屏幕上的视图(登录按钮除外)动画:



![图片](https://raw.githubusercontent.com/CainLuo/CoreAnimationLearn/master/Chapter%202/4.png)



您的工作是处理屏幕上最后一个非动画元素:Log In按钮。

打开ViewController.swift，在viewWillAppear()的底部添加以下代码:

```swift
loginButton.center.y += 30.0
loginButton.alpha = 0.0
```

正如在前一章中所做的那样，您将按钮的起始位置设置在y轴上稍低的位置，并将其alpha值设置为零，以便它从不可见开始。
现在移动到viewDidAppear()并添加以下代码:

```swift
UIView.animate(withDuration: 0.5, delay: 0.5,
usingSpringWithDamping: 0.5, initialSpringVelocity: 0.0, options: [],
animations: {
  self.loginButton.center.y -= 30.0
  self.loginButton.alpha = 1.0
}, completion: nil)
```

这段代码中有两个关键点。

首先，您同时动画了两个不同的属性!这比你想象的要简单，对吧?

其次，您第一次使用了一个新的动画方法:animate(withDuration:delay: usingspringwith阻尼:initialSpringVelocity:optio ns:animation:completion:)。说方法名太快可能会伤到你的舌头!

上面的方法看起来很像您在书的前一章中使用的方法，但是它有两个新参数:

* 使用带阻尼的弹簧:当动画接近最终状态时，它控制应用于动画的阻尼量或减幅。此参数接受0.0到1.0之间的值。接近0.0的值创建一个bouncier动画，而接近1.0的值创建一个僵硬的效果。你可以把这个值看作弹簧的“刚度”。

* initialSpringVelocity:它控制动画的初始速度。值1.0设置动画的初始速度，以在1秒的跨度内覆盖总距离。数值越大越小，动画的速度就会越快或越慢，并会影响弹簧的稳定。不过请注意，初始速度很快就会被spring计算修正，动画总是在持续时间结束时完成。

建立和运行您的项目;现在看看按钮是如何移动的:



![图片](https://raw.githubusercontent.com/CainLuo/CoreAnimationLearn/master/Chapter%202/5.png)



由于您的动画的初始速度为0.0，而中性阻尼为0.5，因此动画看起来不太引人注目。

你应该能够通过尝试不同的速度和阻尼值来美化这个动画。

将持续时间改为3.0，阻尼改为0.1。这只是让你观察你的变化在慢动作的效果，而不是在正常速度。

重新构建和运行您的项目;注意按钮的不透明度如何随着向上移动而变化。这是因为spring行为影响您动画的所有属性;在您的示例中，这将同时影响按钮的垂直位置及其alpha值。

现在将initialSpringVelocity设置为1.0，然后重新构建和运行您的项目:



![图片](https://raw.githubusercontent.com/CainLuo/CoreAnimationLearn/master/Chapter%202/6.png)



您会注意到，当按钮动画化并移动到password字段之外时，它会反弹得更多;这是因为它在运动开始时有更大的动量。

尝试一些不同的阻尼和速度值，直到您理解这些参数的变化如何影响动画的外观和感觉。

完成后，将速度和阻尼的值设置回它们的初始值，如下所示:

```swift
UIView.animate(withDuration: 0.5, delay: 0.5, usingSpringWithDamping:
0.5, initialSpringVelocity: 0.0, options: [], animations: {
  self.loginButton.center.y -= 30.0
  self.loginButton.alpha = 1.0
}, completion: nil)
```

## 动画用户交互

您不必将spring动画限制在视图的初始位置。事实上，响应用户输入的视图动画可以使您的界面变得活跃。在本节中，您将使Log In按钮以动画的方式响应点击。

向login()添加以下代码:

```swift
UIView.animate(withDuration: 1.5, delay: 0.0, usingSpringWithDamping:
0.2, initialSpringVelocity: 0.0, options: [], animations: {
  self.loginButton.bounds.size.width += 80.0
}, completion: nil)
```

上面的动画在一秒半的时间内将按钮的宽度增加了80个点。由于阻尼设置为0.2，按钮也会有一定的弹性。增加边界使框架的左右两边都增大。

建立和运行您的项目;点击这个按钮可以看到你的动画:



![图片](https://raw.githubusercontent.com/CainLuo/CoreAnimationLearn/master/Chapter%202/7.png)



当你点击按钮时，它会以一种水滴状的方式增长和反弹;这是一个向用户提供tap反馈的好方法。

接下来，您将把这个动画与一些更多的弹簧动作结合起来，以真正使按钮具有生命。

在login()的末尾添加以下代码:

```swift
UIView.animate(withDuration: 0.33, delay: 0.0, usingSpringWithDamping:
0.7, initialSpringVelocity: 0.0, options: [], animations: {
  self.loginButton.center.y += 60.0
}, completion: nil)
```

上面的动画在点击时将按钮向下移动60个点。请注意，此动画的持续时间比动画按钮宽度的持续时间短得多。

这是有意的，因为想要的效果是让按钮跳出点击，并反弹一点，一旦它确定到新的垂直位置。

建立和运行您的项目;点击这个按钮，看看它这次是如何响应你的触摸的:



![图片](https://raw.githubusercontent.com/CainLuo/CoreAnimationLearn/master/Chapter%202/8.png)



这看起来真的很好，但你很快就会成为一个动画大师，你知道你可以做得更好!

另一个提供用户反馈的好方法是通过改变颜色。通过动画按钮的backgroundColor属性，可以在按钮移动时为其着色。

将以下代码添加到您添加的最后一个动画中，在动画闭包表达式中:

```swift
self.loginButton.backgroundColor = UIColor(red: 0.85, green: 0.83, blue: 0.45, alpha: 1.0)
```

重新构建和运行您的项目;你会看到按钮同时移动，改变形状和颜色:



![图片](https://raw.githubusercontent.com/CainLuo/CoreAnimationLearn/master/Chapter%202/9.png)



这里还有最后一点反馈:活动指标。Log In按钮应该在网络上启动一个用户身份验证活动，所以最好向用户显示一个活动指示器，让他们知道有一个正在进行的操作。

向上滚动查看viewDidLoad()，并找到进度指示器的现有代码。spinner包含一个UIActivityIndicatorView的实例，可以使用了。您还没有在屏幕上看到它，因为它的alpha值被设置为0.0。

返回login()，并将以下代码添加到最后一个动画闭包表达式中:

```swift
self.spinner.center = CGPoint(
  x: 40.0,
  y: self.loginButton.frame.size.height/2
)
self.spinner.alpha = 1.0
```

此动画将微调微调微调轮向左移动并淡入。这应该足以吸引用户的注意，并让他们知道他们的请求正在处理中。

建立和运行您的项目;看看你漂亮的新动画的最终版本:



![图片](https://raw.githubusercontent.com/CainLuo/CoreAnimationLearn/master/Chapter%202/10.png)



花点时间反思一下你在这里所取得的成就。您已经向按钮视图添加了三个同步动画，以使其变宽、向下移动屏幕和更改颜色。

您还在activity spinner中进行了动画和淡出，activity spinner本身就是button视图的子视图。

所有动画由UIKit自动组合，完美地运行，创造一个流畅的视觉效果。

您无需担心动画的实现细节;相反，你可以专注于设计伟大的动画，感谢UIKit让你的用户惊叹!

## 要点
* 通过创建响应用户操作的动画，您可以为用户交互创建可视化反馈。
* 创建弹簧动画的方式与“标准”动画非常相似——附加的参数是弹簧阻尼和初始速度。
* 结合各种动画(带或不带弹簧)，创造丰富的视觉体验。

## 挑战
### 挑战1:将文本字段动画转换为spring-动画

UIKit中的spring动画api在使用上与标准动画api非常相似。因此，将用户名和密码字段上运行的动画转换为spring动画应该不是什么大问题。
要完成这项挑战，你需要做以下工作:

1. 将usingspringwith阻尼和initialSpringVelocity参数添加到username字段的动画中。使用0.0作为弹簧的初始速度。尝试0.2、0.6和0.9作为弹簧阻尼，并选择对您来说最像是一个愉快而微妙的弹簧效果的值。
2. 使用您选择的阻尼和速度对password字段动画重复相同的操作。



![图片](https://raw.githubusercontent.com/CainLuo/CoreAnimationLearn/master/Chapter%202/11.png)



在本书中，您已经获得了spring动画的坚实基础。你玩这类动画的次数越多，你对阻尼和速度的不同组合进行的实验越多，你就越能在自己的应用程序中为视图设计完美的spring动画。

你准备好进入下一章了吗?在第3章中，你将学习UIKit中的下一种动画类型:转换。

