# Chapter 10: Groups & Advanced Timing

在前一章中，您学习了如何向单层添加多个独立的动画。但是，如果您想让动画同步工作并保持同步，又该怎么办呢?要分别处理所有动画的数学和计时并不有趣。这就是动画组的作用。

本章向您展示了如何使用CAAnimationGroup对动画进行分组，CAAnimationGroup允许您将多个动画添加到一个组中，并同时调整属性，如duration、委托和timingFunction。

对动画进行分组可以简化代码，并确保所有动画同步为一个实体单元。

## CAAnimationGroup



[图片]



首先，您将扩展您的巴哈马航空登录屏幕使用动画组添加一些新的动画登录按钮。

打开本章的启动项目，或者继续上一章的项目和挑战。

打开ViewController.swift并从viewWillAppear()中删除以下代码:

```swift
loginButton.center.y += 30.0
loginButton.alpha = 0.0
```

然后从viewDidAppear()中删除以下代码:

```swift
UIView.animate(withDuration: 0.5, delay: 0.5,
  usingSpringWithDamping: 0.5, initialSpringVelocity: 0,
  animations: {
    self.loginButton.center.y -= 30.0
    self.loginButton.alpha = 1.0
  },
  completion: nil
)
```

…并在其位置上添加以下代码:

```swift
let groupAnimation = CAAnimationGroup()
groupAnimation.beginTime = CACurrentMediaTime() + 0.5
groupAnimation.duration = 0.5
groupAnimation.fillMode = .backwards
```

这段代码创建了一个新的动画组供您使用。CAAnimationGroup继承自CAAnimation，所以您可以使用您已经知道并喜欢的相同属性，比如beginTime、duration、fillMode、delegate和isRemovedOnCompletion。

您将向login按钮添加最后一个动画，用于缩放、旋转和淡入，最后的效果是按钮整齐地放置在屏幕上。

添加第一个动画直接添加下面的代码组动画代码，你刚刚添加:

```swift
let scaleDown = CABasicAnimation(keyPath: "transform.scale")
scaleDown.fromValue = 3.5
scaleDown.toValue = 1.0
```

在上面的代码中，首先是一个非常大的按钮版本，然后在动画的过程中，将其缩小到正常大小。

这里您只指定fromValue和toValue，但是您没有说明动画的持续时间应该是多少，也没有设置动画的fillMode。这些价值观将从何而来?

您可能已经猜到，由于这些值对于组中的所有动画都是相同的，所以您将把它们作为一个整体设置在组上，而不是单独设置在每个动画上。

现在在你刚刚添加的代码下面添加下一个动画的代码:

```swift
let rotate = CABasicAnimation(keyPath: "transform.rotation")
rotate.fromValue = .pi / 4.0
rotate.toValue = 0.0
```

这个动画与前一个类似，但是它动画了图层变换的旋转组件而不是缩放组件。动画开始时，图层以45度角旋转，并将其移动到0度的正常方向。

剩下要添加的就是淡入动画。在刚刚添加的代码行下面再次添加以下代码:

```swift
let fade = CABasicAnimation(keyPath: "opacity")
fade.fromValue = 0.0
fade.toValue = 1.0
```

这是基本的淡入动画，你已经在这本书中见过很多次了。现在添加下面的代码来组合所有动画，并添加到按钮:

```swift
groupAnimation.animations = [scaleDown, rotate, fade]
loginButton.layer.add(groupAnimation, forKey: nil)
```

对于组动画，只需将它们添加到一个数组中，并将该数组指定为组的animation属性的值，就像使用普通的CABasicAnimation一样。

构建并运行您的项目，以查看最终结果:



[图片]



按钮飞进来并按预期旋转，但动画看起来僵硬。在现实生活中，物体在穿过空间时往往会加速。

幸运的是，很容易为动画添加一些真实感。这是一个学习如何在核心动画中使用easing的好机会。

## 动画宽松
你们已经在这本书的第一章看到了实际的缓动，那是关于UIKit动画的。层动画中的缓动在概念上是一样的——只是语法不同。

CAMediaTimingFunction有几个预定义的缓动模式，CAMediaTimingFunctionName包含了这些预定义函数的名称:

* .linear在整个过程中以相同的速度运行动画。
* .easein修改动画，使其开始较慢，结束速度较快



[图片]



* .easeout产生与. easein相反的效果:动画开始时更快，结束时更慢。



[图片]



* .easeineaseout在开始和结束时减慢动画的速度，但在中间部分加快了速度。



[图片]



如果你考虑物体在穿过空间时是如何加速的，你会发现你应该使用一个在接近尾声时加速的简易动画。

在viewDidAppear()中找到初始化groupAnimation的代码段，然后添加以下代码行:

```swift
 groupAnimation.timingFunction = CAMediaTimingFunction(name: .easeIn)
```

这将在整个动画组上设置动画缓动。

建立和运行您的项目;变化是微妙的，但动画看起来更现实和快速:



[图片]



> 注意:虽然这超出了本章的范围，但是您可以构建自己的自定义easing函数。
> 在Apple文档或web上的其他地方详细阅读方便的初始化器`CAMediaTimingFunction(controlPoints:_:_:_:)`这允许您根据三次贝塞尔曲线的控制点来定义缓动函数。

## 更多的时机选择
本章的最后一节将探讨另外四个动画属性，它们允许您控制动画的时间。

### 重复动画
repeatCount允许您重复动画指定的次数。

为了演示这是如何工作的，您将使指令在屏幕上反复显示，而不是只显示一次。在viewDidAppear()中找到代码，在这里您设置了动画的属性，例如持续时间，并将动画重复的次数设置为4:

```swift
flyLeft.repeatCount = 4
```

建立和运行您的项目;您将看到指令总共飞行了四次，之后标签仍然在login按钮下居中。

如果您想要设置总重复时间(以秒为单位)，而不是设置重复次数，请使用repeatDuration属性而不是repeatCount。然而，标签每次重复都会从屏幕上消失，这看起来有点奇怪。您如何创建一个流体，反转动画，使标签飞出屏幕的确切方式，它进入?这比你想象的要容易得多。在设置repeatCount之后添加以下代码:

```swift
flyLeft.autoreverses = true
```

这将运行您的动画反向每次它完成，然后运行在向前运动再次。

建立和运行您的项目;你会看到标签飞进来，然后出来，然后再进来，等等:



[图片]



这很简单，看起来很酷，但是你的动画还是有一点不完美。指令会动画四次，但是标签会直接跳到屏幕中央。

这是因为一个动画周期将标签移到屏幕中心，然后再移出去。因此，当您运行动画四次时，最后一个循环将以标签离开屏幕结束。这就是为什么标签会跳到屏幕中央。你不能运行半个动画周期——你能吗?

将repeatCount从4更改为2.5，如下图所示:

```swift
flyLeft.repeatCount = 2.5
```

现在就构建和运行您的项目;你应该看到动画在你想要的精确位置顺利完成:



[图片]



### 改变动画速度
虽然它们看起来不错，但其中一些动画感觉有点慢。通过设置speed属性，可以独立于持续时间控制动画的速度。

仍然在flyLeft的属性中，在设置autoreverse属性的行后面添加以下代码:

```swift
flyLeft.speed = 2.0
```

即使动画组的持续时间被设置为5秒，动画也将在2.5秒内完成，因为它以两倍的速度运行。

你可以自己设置动画的速度，也可以设置图层的速度。该层与动画遵循相同的计时协议:只需设置该层的速度，就可以影响在该层上运行的所有动画。

在flyLeft之后添加以下代码。您刚刚添加的speed line:

```swift
info.layer.speed = 2.0
```

建立和运行您的项目;左下角的动画运行速度是之前的两倍——但是等等!移动信息标签的动画以四倍的速度运行!发生什么事了?

教训:速度分层次成倍增长。首先在信息层设置速度为2.0，然后将flyLeft的速度也设置为2.0 !最终，信息层以2.0 x 2.0 = 4.0倍于正常速度运行。

只是为了好玩，你可以通过调整顶层视图控制器层的speed属性让屏幕上的一切运行得超级快。在刚刚添加的行下面添加以下行:

```swift
view.layer.speed = 2.0
```

建立和运行您的项目;您将看到表单标题、文本字段，甚至天空中的云都在屏幕上来回移动。

由于动画的乘法因素，信息标签上的淡出动画以4倍的速度运行，而在屏幕上移动信息标签的动画以8倍的速度运行!Wheeeee !

现在你已经对你的项目有了一些乐趣，删除所有的重复和速度调整如下:

```swift
flyLeft.repeatCount = 2.5
flyLeft.autoreverses = true
flyLeft.speed = 2.0
info.layer.speed = 2.0
view.layer.speed = 2.0
```

玩动画的速度，反转和重复是很有趣的，但你的用户可能不会喜欢这样的UI元素在屏幕上快速移动!

但是，如果您确实需要在本地调整动画速度，您现在可以在动画级别以及整个图层上进行调整。

## 要点
* 您可以通过CAAnimationGroup对动画进行分组，并轻松设置组中所有动画共享的公共属性。
* 通过设置动画本身的属性，您可以自定义每个动画的任何共享属性，就像您在前几章中所做的那样。
* 你也可以为你的图层动画使用四个预定义的缓动函数，这些函数你在前面的章节中已经很熟悉了:.linear， . easein， . easeout， . easeinout。

## 挑战
你在这一章学了很多新东西;所以，我不是要你自己去解决一个挑战，而是给你一个免费的通行证，带你看一些你在上面学到的例子。

下面的挑战是可选的，但通过它，您将获得更多的经验与动画组-这从来都不是一件坏事!

### 挑战1:所有表单元素的组动画
在这个挑战中，您将创建一组动画并在表单元素上运行它。因为这段代码将替换现有的表单动画，所以您需要做的第一件事就是删除一些现有的代码。

删除设置表单标题和表单字段初始位置的代码，然后删除将字段动画化到屏幕中心的代码。

在viewWillAppear(_:)中创建一个动画组，就像本章中显示的那样，它组合了以下两个动画:

* 透明度由0.25淡入1.0
* 从图层位置移动。从-view.bounds.size x。宽/ 2 flyRight。toValue = view.bounds.size.width / 2

将两个动画添加到一个动画组中，并在表单标题标签和两个文本字段视图上运行该动画组。

由于希望在组动画完成时启动动画委托方法，因此需要在动画组对象上而不是在单个动画上设置委托和键。

在将其添加到每个文本字段之前，不要忘记使用setValue(_:forKey:)在动画上设置名称和图层键，以便正确调用委托方法。

此外，您将需要调整您的动画组对象的beginTime参数，以便像以前一样为正在动画的不同层提供正确的延迟。

通过这项挑战，您可以获得更多与团队和代表一起工作的宝贵经验。现在，您已经准备好进入下一章，并为层动画添加一些弹性。