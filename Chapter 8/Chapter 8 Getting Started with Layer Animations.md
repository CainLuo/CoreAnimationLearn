# Chapter 8: Getting Started with Layer Animations

层动画的工作原理很像视图动画;您只需在定义的一段时间内，在起始值和结束值之间对属性进行动画处理，并让Core Animation处理中间的呈现。

然而，图层具有比视图更多的可动画属性;这给了你很多选择和灵活性，当涉及到设计你的效果;许多专门化的CALayer子类添加了可以在动画中使用的其他属性。

本章将介绍CALayer和Core Animation的基础知识。你会有在图层中处理动画的感觉;您将学习如何移动图层，淡入淡出图层，并创建与您使用UIKit创建的动画类似的动画。

## 可以做成动画属性
CALayer中的一些动画属性直接对应于您在前几章中使用过的视图属性，例如框架、位置和不透明度。在本章中，您将看到层动画中使用的熟悉的和新的动画属性。

您将重新创建一些早期的视图动画，但是使用了图层，因此您可以画出相似点，并亲自查看相似点在哪里结束——以及新的可能性在哪里开始。

### 位置和大小

[图片]



对一个层的位置、大小或转换进行动画处理同样会影响该层中包含的任何视图，就像直接对视图本身进行动画处理一样。

* 边界:修改它，使层的边界框具有动画效果。
* 位置:修改它，使该层在其母层中的位置具有动画效果。你可以动画位置。x或位置。如果你想单独控制一个轴的运动。
* 变换:修改它来移动、缩放和旋转图层。你甚至可以在三维空间中动画图层，这是你不能单独用视图做的。您将在第四节“3D动画”中学习3D层转换。

### 边境

[图片]



你可以很容易地动画一个图层的边界，以改变它的颜色，宽度和角半径:

* border color:修改它来改变边框颜色。
* border width:修改它来增加或缩小边框的宽度。
* 角半径:修改这个来改变图层圆角的半径。

### 影子

[图片]



你可以动画图层阴影的所有方面:

* 阴影偏移:修改这个，使阴影看起来更接近或更远离图层。
* 阴影不透明度:修改这个使阴影淡入或淡出。
* 阴影路径:修改这个来改变图层阴影的形状。你可以创建不同的3D效果，让你的图层看起来像漂浮着不同的阴影形状和位置。
* 阴影半径:修改这个来控制阴影的模糊;这在模拟视图朝向或远离投射阴影的表面的移动时特别有用。

### 内容

[图片]

最后，有一些属性可以控制图层内容的渲染方式:

* 内容:修改此属性，将原始TIFF或PNG数据指定为层内容。
* 蒙版:修改它，以建立形状或图像，您将使用蒙版层的可视内容;在第13章“形状和蒙版”中，您将使用这个属性创建一些非常酷的效果。
* 不透明度:修改它，使图层内容的透明度具有动画效果。

请记住，这只是您可以动画的属性列表的一部分;CALayer的子类通常还有其他可以动画化的属性。

上面列出的属性足以让您开始;现在是开始使用图层属性制作第一个动画的时候了。

## 你的第一层动画
您将从第3章“过渡”的末尾开始完成巴哈马航空登录屏幕项目。与往常一样，您可以在前面的工作基础上继续构建，或者打开本章中包含的启动项目。

构建并运行您的项目，可以看到熟悉的巴哈马航空登录屏幕:



[图片]



您的工作是删除现有的视图动画并用基于层的动画逐个替换它们。

打开ViewController.swift并找到viewWillAppear()。

删除下面将标题移出屏幕边界的行:

```swift
heading.center.x -= view.bounds.width
```

不再需要执行这个操作，因为您可以在层动画中指定起始值和结束值。

接下来，向下滚动到viewDidAppear()并删除移动标题的动画调用，如下所示:

```swift
UIView.animate(withDuration: 0.5) {
  self.heading.center.x += self.view.bounds.width
}
```

建立和运行您的项目;查看动画屏幕，检查表单标题是否不再动画:



[图片]



现在您已经删除了旧的视图动画代码，是时候添加一些图层动画了!找到viewWillAppear()，并在方法顶部的super调用下面添加以下代码:

```swift
let flyRight = CABasicAnimation(keyPath: "position.x")
flyRight.fromValue = -view.bounds.size.width/2
flyRight.toValue = view.bounds.size.width/2
flyRight.duration = 0.5
```

核心动画中的动画对象只是简单的数据模型;您可以创建模型的实例并相应地设置它的数据属性。

CABasicAnimation的一个实例描述了一个潜在的层动画:您可以选择现在运行它，稍后再运行，或者根本不运行它。由于动画没有绑定到特定的层，您可以在其他层上重用动画，并且每个层将独立运行动画的副本。在动画模型中，可以将要动画的属性指定为keypath参数;这很方便，因为你总是在图层中创建动画。

这里，你只是在动画位置的x分量。Core Animation可以方便地公开位置、边界和转换的各个成员，这样就可以分别对它们进行动画处理。

接下来，为在keypath上指定的属性设置fromValue和toValue。在本例中，您希望它从屏幕外的左侧开始，并最终位于屏幕的中心。

最后，动画时长的概念没有改变;这里将持续时间设置为0.5秒。

现在动画都设置好了，你可以把它添加到你的应用程序的一个图层中，看看它看起来怎么样。添加下面的代码行你刚刚添加到添加你的动画到你的标题层:

```swift
heading.layer.add(flyRight, forKey: nil)
```

add(_:forKey:)复制动画对象，并告诉Core animation在图层上运行它。关键参数仅供您使用;如果以后需要更改或停止动画，它允许您标识动画。

建立和运行您的项目;您将看到表单标题移动到屏幕中央，如下所示:



[图片]



正如预期的那样，这个图层——以及它所包含的视图——顺利地移动到合适的位置。这显示视图及其支持层的绑定有多紧密。

> 注意:CGRect或CATransform3D等结构的动画并不像上面使用对象值那样简单。您将在第12章“关键帧动画和结构属性”中看到如何对结构体进行动画。

现在您已经掌握了基本知识，事情只会变得更有趣!

## 更精细的图层动画
您已经处理了登录屏幕上的标题层;您的下一个任务是处理username字段。

滚动到viewWillAppear()并删除以下行:

```swift
username.center.x -= view.bounds.width
```

然后从viewDidAppear()的用户名字段中删除以下视图动画:

```swift
UIView.animateWithDuration(0.5, delay: 0.3,
  usingSpringWithDamping: 0.6, initialSpringVelocity: 0,
  animations: {
    self.username.center.x += self.view.bounds.width
  },
  completion: nil
)
```

在快速浏览并盲目复制和粘贴代码为username字段创建CABasicAnimation之前，请考虑以下两个事实:

* CABasicAnimation对象只是一个数据模型，它不绑定到任何特定的层。
* add(_:forKey:)复制动画对象。

事实证明，您可以简单地从标题层获取动画，如果需要的话稍微调整一下动画，然后重用它将用户名字段动画到屏幕上。在viewWillAppear()中删除上述代码的位置添加以下代码:

```swift
username.layer.add(flyRight, forKey: nil)
```

构建并运行您的项目，以查看重用图层动画的效果:



[图片]



动画运行时，标题和用户名字段会滑到屏幕上，就像那些不受欢迎的同步杀人机器人一样。您需要重新创建原始动画中的时间偏移量。

添加以下行之前的行，您添加的flyRight动画到您的用户名层:

```swift
flyRight.beginTime = CACurrentMediaTime() + 0.3
```

动画的beginTime属性设置动画应该启动的绝对时间;在本例中，您使用CACurrentMediaTime()获得当前时间，并将所需的延迟(以秒为单位)添加到其中。

再次构建和运行你的应用程序，看看事情是怎样的;它出现在屏幕中央，因为它是在Interface Builder中设计的，并在0.3秒后开始动画。到底发生了什么事?

现在是学习另一个图层动画属性fillMode的时候了;下面是这个属性如何工作的几个例子。

### 使用fillMode
fillMode属性允许您在动画序列的开始和结束时控制动画的行为。

常数CAMediaTimingFillMode。remove是fillMode的默认值。这将在定义的beginTime开始动画-或立即，如果你没有设置beginTime -并删除在动画过程中所做的更改时，动画完成:



[图片]



这就是您在本章到目前为止使用的方法。除了删除，你还可以在动画中使用其他三个选项:

#### 向后
CAMediaTimingFillMode。无论动画的实际开始时间如何，后退都会在屏幕上立即显示动画的第一帧，并在稍后的时间启动动画。



[图片]



#### 转发
CAMediaTimingFillMode。forward和往常一样播放动画，但保留动画的最终帧，直到删除动画:



[图片]



除了设置CAMediaTimingFillMode。向前，您将需要对层做一些其他的更改，以使最后一帧“坚持”。您将在本章稍后的部分了解这一点。



#### 双向

CAMediaTimingFillMode。两者都是向前和向后的组合;正如你所期望的，这使得动画的第一帧立即出现在屏幕上，并在动画完成时保留屏幕上的最后一帧:



[图片]



要修复前面发现的问题，可以同时使用这两种方法。

在你设置flyRight (fromValue, toValue, duration等)的地方添加以下代码，然后在你添加到图层之前

```swift
flyRight.fillMode = .both
```

建立和运行您的项目;您将看到用户名字段没有首先出现，动画只在0.3秒的延迟之后才开始。此外，当动画完成时，字段仍然保持在适当的位置。

您现在可以以类似的方式激活密码字段。从viewWillAppear()中删除以下行:

```swift
password.center.x -= view.bounds.width
```

然后在viewDidAppear()中找到并删除以下代码:

```swift
UIView.animate(withDuration: 0.5, delay: 0.4, options: .curveEaseOut,
animations: {
  self.password.center.x += self.view.bounds.width
}, completion: nil)
```

…然后用viewWillAppear()中的以下代码替换它:

```swift
flyRight.beginTime = CACurrentMediaTime() + 0.4
password.layer.add(flyRight, forKey: nil)
```

建立和运行您的项目;你会看到所有三层都飞了进来，密码字段只比用户名字段晚十分之一秒到达:



[图片]



到目前为止，您的动画恰好结束于表单元素最初在Interface Builder中的位置。很多时候，情况并非如此。在本章的下一节中，您将发现如何处理层在不同位置结束的情况!

## 动画与真实内容
当你给一个文本框设置动画时，你实际上并没有看到这个字段本身被动画化;相反，您将看到它的缓存版本，称为表示层。一旦动画完成并再次显示原始层，表示层就从屏幕上删除。

首先，请记住您将文本字段设置为viewWillAppear(_:)中的屏幕外位置:



[图片]



当动画开始时，一个预渲染的动画对象代替了字段，原始文本字段暂时隐藏:



[图片]



您不能点击动画字段、输入任何文本或使用任何其他特定的文本字段功能，因为它不是真正的文本字段，只是一个“幻影”可见表示。

一旦动画完成，它就从屏幕上消失，原来的文本字段也不再隐藏。文本字段就在您离开它的地方:屏幕外的左侧!



[图片]



要解决这个难题，您需要使用另一个CABasicAnimation属性:isRemovedOnCompletion。

将fillMode设置为这两个参数，既可以指示动画在完成后留在屏幕上，也可以显示动画开始前的第一帧。为了完成效果，您需要相应地设置removedOnCompletion;两者的结合将使动画在屏幕上可见。

在设置fillMode之后，将以下行添加到viewWillAppear():

```swift
flyRight.isRemovedOnCompletion = false
```

默认情况下isRemovedOnCompletion为true，所以动画一完成就消失了。将其设置为false，并将其与适当的fillMode相结合，可以使动画在屏幕上保持可见。

现在就构建和运行您的项目;你应该看到，一旦动画完成，所有的元素都像预期的那样留在屏幕上:



[图片]



成功!现在点击用户名输入你的用户名-哦，等等。还记得前面关于实际文本字段和表示层之间的区别的说明吗?您无法对这个文本字段的预呈现图像执行任何操作。

要完成预期的效果，需要删除动画并在其位置显示真实的文本字段。



### 更新图层模型
一旦你从屏幕上移除一个图层动画，这个图层就会返回到它当前的位置和其他属性值。这意味着您通常需要更新层的属性来反映动画的最终值。

从项目中删除以下行:

```swift
flyRight.isRemovedOnCompletion = false
```

尽管您知道isRemovedOnCompletion在设置为false时是如何工作的，但是尽可能地避免它。将动画留在屏幕上会影响性能，因此您将自动删除它们并更新原始层的位置。

接下来，在viewWillAppear()中找到将动画添加到username的代码行，并在其后面添加以下行:

```swift
username.layer.position.x = view.bounds.size.width/2
```

接下来，添加密码字段的代码。找到将动画添加到密码中的行，并在其后面添加以下行:

```swift
password.layer.position.x = view.bounds.size.width/2
```

这将把实际的图层设置在它们所在的屏幕中间。

建立和运行您的项目;哦哦。田野已经完全停止了活动!发生什么事了?

您在一段时间之前删除了set fromValue的行，但是上面的代码更新了主层的位置;这将导致动画从屏幕的中心开始。要解决这个问题，在初始化flyRight并将其设置为toValue之后添加以下行:

```swift
flyRight.fromValue = -view.bounds.size.width/2
```

再次构建并运行您的项目，这一次您将看到字段按预期动画化。

> 注意:如果在模拟器中单击文本字段时没有显示键盘，可以通过导航菜单到硬件\键盘\切换软件键盘手动激活它。

如果可能，在Interface Builder中使用它们的最终值设计层，并使用fromValue作为初始值和中间值。这减少了保持模型层和表示层同步的复杂性。

## 最佳实践
哇，这是一个很长的章节!你尝试了大量不同的图层动画技术，这只是开始!

这时你可能会觉得有点不知所措，问自己“我应该使用fillMode吗?”我应该删除我的动画吗?我如何更新我的图层，使动画顺利完成?”

经验法则:删除动画并考虑永远不要使用fillMode，除非你想要达到的效果是不可能的。fillMode使UI元素失去交互性，也使屏幕不能反映层对象中的实际值。

在一些罕见的情况下，当你动画非互动的视觉元素填充模式将节省您的培根;您将在第16章“复制动画”中阅读更多关于这方面的内容。

关于更新你的图层属性:考虑总是在你把动画添加到你的图层后立即这样做。有时您可能会在初始动画值和最终动画值之间得到奇怪的flash。

在这种情况下，在添加动画之前，尝试将您的layer属性更新为最终的动画值。

## 要点
* 在创建UI动画时，图层动画提供了更多的选项。与视图动画不同，您可以使许多额外的处理动画，如角半径、阴影、边框宽度和颜色、边框样式等。

* CABasicAnimation是一个基本的动画模型类，你可以用它来描述你想要的动画，你可以通过调用CALayer来进行渲染。添加(_ forKey:)。

* 由于层动画是添加到层中的Core Animation复制的数据模型，所以您可以重用相同的模型实例来创建许多类似的动画，甚至可以在将其添加到不同的层之间调整它的一些属性。

## 挑战
你在这一章涵盖了很多内容;如果您真的想测试您是否保留了每个部分中涵盖的所有概念，请接受下面的挑战。

如果您需要一些帮助，请参阅各个部分，但是如果您已经完成了本章中的练习，那么您完全有能力完成本章中的三个挑战!

### 挑战1:用图层动画淡入云层

在这个挑战中，你将用图层动画代替第1章“从视图动画开始”中的UIKit云动画。

如果你需要一个食谱来遵循，下面的步骤应该给你一个很好的起点:

1. 从viewDidAppear()中删除四个以cloud1、cloud2、cloud3和cloud4淡出的UIKit动画。
2. 从viewWillAppear()中删除将云的alpha属性设置为0.0的四行代码。
3. 在viewWillAppear()的底部，创建一个不透明度为键路径的CABasicAnimation。
4. 将from值设置为0.0,toValue设置为1.0,duration设置为0.5。您需要将fillMode设置为. reverse，以便在应用程序启动时隐藏云。
5. 将动画的beginTime设置为CACurrentMediaTime() + 0.5并将动画添加到cloud1。
6. 将动画的beginTime设置为CACurrentMediaTime() + 0.7并将动画添加到cloud2。
7. 将动画的beginTime设置为CACurrentMediaTime() + 0.9并将动画添加到cloud3。
8. 将动画的beginTime设置为CACurrentMediaTime() + 1.1并将动画添加到cloud4。

最终的结果应该重新创建初始的云过渡层动画如下图所示:



[图片]



### 挑战2:动画色彩
在这个挑战中，您将重新创建登录按钮着色动画。首先，删除旧的UIKit代码，如下所示:

```swift
self.loginButton.backgroundColor = UIColor(red: 0.85, green: 0.83, blue: 0.45, alpha: 1.0)
```

然后向下滚动到resetForm()并删除以下行:

```swift
self.loginButton.backgroundColor = UIColor(red: 0.63, green: 0.84, blue: 0.35, alpha: 1.0)
```

在ViewController.swift中创建一个新的顶级函数(例如，一个顶级函数将位于类主体之外;添加到延迟函数下面):

```swift
func tintBackgroundColor(layer: CALayer, toColor: UIColor)
```

在这个新方法中，创建一个基本的动画，并运行它的图层参数，记住以下关键要求:

* 激活backgroundColor属性。
* 将fromValue设置为当前背景颜色:layer.backgroundColor。
* 将toValue设置为toColor.CGColor;Core Animation对颜色使用CGColor值。
* 设置动画持续时间为1.0秒。
* 添加动画到图层。
* 不要忘记设置图层本身的backgroundColor属性，以便保留最终的动画颜色。

现在tintBackgroundColor已经完成，您可以在登录按钮上使用它。为此，在login()的底部添加以下代码:

```swift
let tintColor = UIColor(red: 0.85, green: 0.83, blue: 0.45, alpha: 1.0)
tintBackgroundColor(layer: loginButton.layer, toColor: tintColor)
```

建立和运行您的项目;你会看到按钮改变它的颜色，只要你点击它:



[图片]



现在在resetForm()中找到animate(…)的调用，并用以下代码替换completion: nil:

```swift
completion: {_ in
  let tintColor = UIColor(red: 0.63, green: 0.84, blue: 0.35, alpha: 1.0)
  tintBackgroundColor(layer: self.loginButton.layer, toColor: tintColor)
}
```

这将使按钮在动画完成后变回绿色。另外，由于您的新函数tintBackgroundColor是一个顶级函数，所以您可以在您的项目中重用它!

### 挑战3:动画角落半径
在这个挑战中，您不会重新创建一个现有的视图动画;相反，您将使层的特定属性cornerRadius具有动画效果。就像您在上面的挑战2中所做的一样，在ViewController.swift中创建以下新的顶级函数:

```swift
func roundCorners(layer: CALayer, toRadius: CGFloat)
```

该方法将获取一个层参数并在其上运行一个基本动画，该动画将角半径从当前值动画到提供的toRadius。
简单地重复挑战2中的步骤，但要遵循以下要求:

* 动画的角半径属性。
* 将value设置为toRadius。
* 将动画持续时间设置为0.33秒。
* 添加动画到图层。
* 不要忘记设置图层的角半径。

完成roundCorners()后，在login()的底部添加以下行:

```swift
roundCorners(layer: loginButton.layer, toRadius: 25.0)
```

当你点击这个按钮时，它应该是圆形的。要逆转身份验证过程完成时的效果，请在调用tintBackgroundColor后将以下代码添加到resetForm():

```swift
roundCorners(layer: self.loginButton.layer, toRadius: 10.0)
```

建立和运行您的项目;你会看到按钮出现与它的初始颜色和角半径:



[图片]



现在点击按钮可以看到它的形状和颜色的变化:



[图片]



一旦所有动画完成，按钮将返回其初始颜色和角半径。

这一章就讲到这里;现在您已经对如何创建基本图层动画有了一个坚实的了解，这是在下一章处理动画键和委托方法的完美起点!