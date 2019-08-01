# Chapter 1: Getting Started with View Animations

苹果的UIKit框架，用于创建丰富的iOS应用程序用户界面，提供了多种api，你可以用来在屏幕上动画视图。UI动画并不是电影或电视中通常所说的“帧动画”，而是一组预定义的操作，比如淡入、移动和调整屏幕上的视图大小。
在本章和伴随的项目中，您将学习如何做以下事情:

* 为一个很酷的动画搭建舞台。
* 创建移动和淡入动画。
* 调整动画的缓动。
* 反转和重复动画。

有很多材料要讲，但我保证会很有趣。你准备好接受挑战了吗?

## 你的第一个动画
打开本章Resources文件夹中的starter项目。在Xcode中构建和运行项目;您将看到一个虚构的航空公司应用程序的登录屏幕，如下所示:



[图片]



这款应用目前并没有太多功能;它只显示了一个带有标题的登录表单、两个文本字段和底部的一个大的友好按钮。

还有一张不错的背景图片和四朵云。这些云已经连接到名为cloud1 through cloud4的代码中的出口变量。

打开ViewController.swift，看看里面。在文件的顶部，您将看到所有连接的outlet和类变量。再往下，viewDidLoad()中有一小段代码用于初始化一些UI。这个项目已经为您准备好了，您可以投入其中并进行一些调整!

介绍到此为止——您无疑已经准备好尝试一些代码了!

您的第一个任务是在用户打开应用程序时将表单元素动画化到屏幕上。由于该表单在应用程序启动时是可见的，所以您必须在视图控制器出现之前将其移出屏幕。

将以下代码添加到viewWillAppear():

```swift
heading.center.x  -= view.bounds.width
username.center.x -= view.bounds.width
password.center.x -= view.bounds.width
```

这将每个表单元素放置在屏幕的可见边界之外，如下所示:



[图片]



由于上面的代码在视图控制器出现之前执行，所以看起来这些文本字段一开始就不存在。

建立和运行你的项目，以确保你的字段真正出现在屏幕外，正如你的计划:



[图片]



完美-现在你可以通过一个令人愉快的动画把这些形式元素动画回到它们原来的位置。

在viewDidAppear()的末尾添加以下代码:

```swift
UIView.animate(withDuration: 0.5) {
  self.heading.center.x += self.view.bounds.width
}
```

要将标题动画到视图中，你调用UIView类方法animate(带有duration:animation:)。动画立即启动，动画时间超过半秒;您可以通过代码中的第一个方法参数设置持续时间。

就这么简单;你在animation闭包中对视图所做的所有改变都会被UIKit动画化。

建立和运行您的项目;你应该看到标题整齐地滑到这样的地方:



[图片]



这为您在其他表单元素中设置动画设置了舞台。

既然animate(withDuration:animation:)是一个类方法，您就不局限于动画一个特定的视图;实际上，您可以在动画闭包中对任意多个视图进行动画处理。

添加以下行到动画关闭:

```swift
self.username.center.x += self.view.bounds.width
```

重新构建和运行您的项目;查看用户名字段的位置:



[图片]



看到两个视图一起动画是非常酷的，但是您可能注意到在相同的距离和相同的持续时间内对两个视图进行动画看起来有点僵硬。只有杀人机器人才会这样绝对同步地移动!

如果每个元素都独立于其他元素移动，可能在动画之间会有一点延迟，不是很酷吗?

首先删除刚才添加的动画用户名行:

```swift
self.username.center.x += self.view.bounds.width
```

然后在viewDidAppear()的底部添加以下代码:

```swift
UIView.animate(withDuration: 0.5, delay: 0.3, options: [],
  animations: {
    self.username.center.x += self.view.bounds.width
  },
  completion: nil
)
```

这次使用的类方法看起来很熟悉，但是它有更多的参数可以让你自定义动画:

* withDuration:动画的持续时间。
* 延迟:UIKit在动画开始前等待的时间。
* 选项:让您自定义动画的许多方面。稍后您将了解关于这个参数的更多信息，但是现在您可以传递一个空数组[]来表示“没有特殊选项”。
* 动画:提供动画的闭包表达式。
* 完成:动画完成时执行的代码闭包。当您希望执行一些最终的清理任务或一个接一个地链接动画时，这个参数经常派上用场。

在上面添加的代码中，您将延迟设置为0.3，使动画比标题动画晚一点点启动。
建立和运行您的项目;现在的合成动画看起来怎么样?

啊——看起来好多了。现在您需要做的就是在password字段中设置动画。



[图片]



在viewDidAppear()的底部添加以下代码:

```swift
UIView.animate(withDuration: 0.5, delay: 0.4, options: [],
  animations: {
    self.password.center.x += self.view.bounds.width
  },
  completion: nil
)
```

这里您主要模仿了username字段的动画，只是延迟稍微长一些。
再次构建并运行您的项目，以查看完整的动画序列:



[图片]



这就是你用UIKit动画在屏幕上动画视图所需要做的一切!
这只是开始-你将学习一些更可怕的动画技术在这一章的其余部分!

## 可以做成动画属性

既然您已经看到了动画是多么简单，那么您可能很想了解如何使视图具有动画效果。

本节将概述UIView的可动画属性，然后指导您在项目中研究这些动画。

不是所有视图属性都可以动画化，但是所有视图动画，从最简单的到最复杂的，都可以通过动画化视图上的属性子集来构建，这些属性本身就是动画，如下面的小节所述。

### 位置和大小



[图片]



您可以对视图的位置和帧进行动画处理，以使其增长、收缩或移动，就像您在上一节中所做的那样。下面是你可以用来修改视图位置和大小的属性:

* bounds:使该属性具有动画效果，以便在视图的框架内重新定位视图的内容。
* frame:将此属性设置为动画，以移动和/或缩放视图。
* center:当你想要将视图移动到屏幕上的新位置时，动画这个属性。

不要忘记在Swift中，一些UIKit属性，比如size和center是可变的。这意味着您可以通过更改中心来垂直移动视图。也可以通过减小frame.size.width来缩小视图。

### 外观

[图片]



您可以通过为视图的背景着色或使视图完全或半透明来更改视图内容的外观。

* background color:改变视图的这个属性，让UIKit随着时间逐渐改变背景颜色。
* alpha:更改此属性以创建淡入和淡出效果。

### 转换

[图片]

转换修改视图的方法与上面的方法非常相似，因为您还可以调整大小和位置。

* transform:在动画块中修改此属性，使视图的旋转、缩放和/或位置具有动画效果。

这些是引线下的仿射变换，它们更强大，允许您描述缩放因子或旋转角度，而不需要提供特定的边界或中心点。
这些看起来是非常基本的构建块，但是您将会对即将遇到的复杂动画效果感到惊讶!

## 动画选项

回顾您的动画代码，您总是将[]传递给options参数。选项允许你自定义UIKit如何创建你的动画。您只调整了动画的持续时间和延迟，但是您可以对动画参数进行更多的控制。

下面是在UIView中声明的选项列表。AnimationOptions设置类型，您可以以不同的方式组合使用，以便在动画中使用。

### 重复

你将首先看看以下两个动画选项:

•.repeat:包含此选项可以使动画永久循环。
•.autoreverse:仅在.repeat中包含此选项;这个选项重复播放动画向前，然后反向播放。

修改密码字段viewDidAppear()的动画代码，使用.repeat选项，如下所示:

```swift
UIView.animate(withDuration: 0.5, delay: 0.4,
  options: .repeat,
  animations: {
    self.password.center.x += self.view.bounds.width
  },
  completion: nil
)
```

构建并运行您的项目，以查看更改的效果:



[图片]



表单标题和用户名字段会飞进来，并在屏幕中央定居下来，但是密码字段会从屏幕外的位置永远保持动画状态。

修改上面修改过的代码，在options参数中同时使用.repeat和.autoreverse，如下所示:

```swift
UIView.animate(withDuration: 0.5, delay: 0.4,
  options: [.repeat, .autoreverse],
  animations: {
    self.password.center.x += self.view.bounds.width
  },
  completion: nil
)
```

注意，如果希望启用多个选项，需要使用set语法列出所有用逗号分隔的选项，并将列表括在方括号中。

> 注意:如果您只需要一个选项，Swift允许您省略方括号，这是一种方便。不过，您仍然可以包含它们，以防将来添加更多选项。这意味着没有选择的余地。对于单个选项重复]，并且[。重复，.autorepeat]用于多个选项。

重新构建和运行您的项目;这次密码字段无法决定是否继续显示在屏幕上!

### 动画宽松
在现实生活中，事情不会突然开始或停止移动。像汽车或火车这样的物理物体会慢慢加速，直到达到目标速度，除非撞到砖墙，否则它们会逐渐减速，直到在最终目的地完全停下来。

下图详细说明了这个概念:



[图片]



为了让你的动画看起来更真实，你可以在开始时使用相同的效果，在结束前放慢速度，一般来说被称为放松和放松。

你可以从四种不同的放松选项中选择:

* .curvelinear:该选项不应用于动画的加速或减速。在本书中，您唯一一次使用这个选项是在第三章“转换”的最后一个挑战中。
* .curveeasein:这个选项将加速应用到动画的开始。
* .curveeaseout:该选项将减速应用到动画的末尾。
* .curveeaseinout:这个选项在动画开始时应用加速，在动画结束时应用减速。

为了更好地理解这些选项如何为动画添加视觉效果，您将尝试项目中的一些选项。

再次修改动画代码为您的密码字段与一个新的选项如下:

```swift
UIView.animate(withDuration: 0.5, delay: 0.4,
  options: [.repeat, .autoreverse, .curveEaseOut],
  animations: {
    self.password.center.x += self.view.bounds.width
  },
  completion: nil
)
```

建立和运行您的项目;注意，在返回到屏幕左侧之前，字段减速是多么平稳，直到它到达最右边的位置:



[图片]



这看起来更自然，因为这是你在现实世界中期望事物运动的方式。

现在，试试相反的方法。当字段仍然在屏幕外时，通过修改与上面相同的代码，将.curveEaseOut选项更改为.curveEaseIn，从而使动画变得容易:

```swift
UIView.animate(withDuration: 0.5, delay: 0.4,
  options: [.repeat, .autoreverse, .curveEaseIn],
  animations: {
    self.password.center.x += self.view.bounds.width
  },
  completion: nil
)
```

建立和运行您的项目;观察磁场如何以机器人的力量从最右边的位置弹回来。这看起来不自然，也不像之前的动画那么赏心悦目。

最后，试一试。curveeaseinout是一个默认的缓动函数UIKit应用于你的动画。

您已经了解了各种动画选项如何影响您的项目，以及如何使动作看起来流畅和自然。

在继续之前，将你一直在玩的代码段的选项改为[]:

```swift
UIView.animate(withDuration: 0.5, delay: 0.4, options: [],
  animations: {
    self.password.center.x += self.view.bounds.width
  },
  completion: nil
)
```

既然您已经了解了基本动画的工作原理，那么您已经准备好处理一些更令人眼花缭乱的动画技术。

## 要点

* 使用UIView.animate(…)的变体之一创建动画。

* 在animation closure中设置所需动画的最终状态，UIKit将自动在当前和最终状态之间创建一个平滑的动画。

* 你可以通过提供UIView来定制你的动画。AnimationOptions值，用于设置缓动、重复和自动反转属性。

## 挑战
如果这一章是你第一次在iOS中创建视图动画，你的头可能会有点晕。不过，不要担心，因为无论你最初的技能如何，只要几章，你就能很好地掌握动画。不过现在，有一个非常简单的挑战等待着你，你将创建一个自己的动画。

### 挑战1:在云中消失
在ViewController中有四个outlet: cloud1、cloud2、cloud3和cloud4。您的任务是在应用程序启动时淡入这些内容。

你几乎可以决定你的解决方案的具体形式，但这里有一个你需要遵循的基本步骤列表:

1. 将viewWillAppear()中的所有四个云视图的alpha属性设置为0.0。
2. 在viewDidAppear()中，分别调用四个animate(使用duration:delay:options:animation:completion:)。如果您对所有四个动画都使用0.5的持续时间，并且延迟分别为0.5、0.7、0.9和1.1，那么您将得到一个漂亮的效果。
3.在每个动画中，闭包将各自云视图的alpha值更改为1.0。这将使云淡入。

当你运行这个项目，你应该看到一个很好的过渡效果，动画云，一个接一个:



[图片]



屏幕上的所有视图都应该具有良好的动画效果。好近……登录按钮没有动画!别担心，你会在下一章解决这个问题的。