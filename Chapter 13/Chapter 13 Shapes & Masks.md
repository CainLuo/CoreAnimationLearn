# Chapter 13: Shapes & Masks

本章标志着本书这一部分的一个转变：你不仅要开始使用不同的示例项目，而且还要使用多层效果，创建看似与每个物理交互的图层动画。 其他，并在动画运行时在形状之间变换。

如果这听起来有很多东西，那就回想一下你在前几章中用相对较少的代码创建的精美动画吧！

本章中的形状将由CAShapeLayer处理，这是一个CALayer子类，可以让您在屏幕上绘制各种形状，从非常简单到非常复杂：



[图片]



您可以在屏幕上绘制CALayer CGPath，而不是接受绘图说明。 这很方便，因为Core Graphics已经为构建CGPath形状定义了非常广泛的绘图指令API。

如果您对UIBezierPath更熟悉，可以使用它来定义形状，然后使用其cgPath属性来获取其Core Graphics表示。 您将在本章后面尝试一下。

创建所需形状后，可以将此类属性设置为笔触颜色，填充颜色和笔触虚线模式。

当然，到现在为止你可能会问“......但是我可以为这些属性设置动画吗？”是的，你可以：

* path：将图层的形状变形为不同的形状。
* fillColor：将形状的填充色调更改为其他颜色。
* lineDashPhase：围绕形状创建选框或“行进蚂蚁”效果。
* lineWidth：增大或缩小形状笔划线的大小。

绘制形状时可以使用另外两个可动画的属性; 你将在第15章“中风和路径动画”中了解这些内容。

本章的项目模拟了搜索在线对手的战斗游戏的起始屏幕。 您将模拟一些在线通信并添加动画以显示通信状态。

到本章结束时，该项目看起来很像下面的屏幕：



[图片]



本章旨在向您展示如何在您在现实生活中使用的常见项目的上下文中为上面讨论的新属性设置动画。 这将需要一些额外的工作，但我知道你会喜欢骑！

## 完成头像视图
打开本章的入门项目，选择Main.storyboard并查看本章中将要使用的用户界面：



[图片]



项目设置相当简单：单个视图控制器显示漂亮的背景图像，一些标签，“再次搜索”按钮和两个头像图像，其中一个将是空的，直到应用程序“找到”对手。

这两个头像都是AvatarView类的一个实例。 在本章的这一部分中，您将在学习AvatarView的工作原理时快速完成类代码的编写。

打开AvatarView.swift并查看didMoveToWindow（），您将在其中构建头像视图的以下元素：

* photoLayer：头像的图像层。
* circleLayer：用于绘制圆的形状图层。
* maskLayer：用于绘制蒙版的另一个形状图层。
* label：显示玩家姓名的标签。

您将这些层叠在一起以构建复合化身视图，如下所示：



[图片]



上面的组件已经存在于项目中，但尚未添加到视图中 - 这是您的第一个任务。 将以下代码添加到didMoveToWindow（）：

```swift
photoLayer.mask = maskLayer
```

这简单地用**maskLayer**中的圆形掩模掩盖上面的方形图像。 构建并运行您的项目以查看事物的外观; 您还可以通过**@IBDesignable**在故事板中看到更改：

现在将边框图层添加到didMoveToWindow（）中的头像视图图层：

```swift
layer.addSublayer(circleLayer)
```

这会将圆形图层添加到头像中，可以很好地构图：



[图片]



掩模层和帧层都是CAShapeLayer的实例; 当你在下一部分中为它们设置动画时，你会利用这个事实。

还有一件要添加的东西 - 玩家名称标签。 将以下代码添加到didMoveToWindow（）：

```swift
addSubview(label)
```

这样包装了头像视图：



[图片]



现在您已准备好添加一些动画！

## 创建反弹动画
您创建的第一个动画将使您看起来好像两个化身在您的项目“搜索”对手时互相反弹。



[图片]



打开ViewController.swift并将以下行添加到viewDidAppear（）：

```swift
searchForOpponent()
```

这个方法开始你的搜索对手动画。

将以下代码添加到ViewController以创建此方法的第一次迭代：

```swift
func searchForOpponent() {
  let avatarSize = myAvatar.frame.size
  let bounceXOffset: CGFloat = avatarSize.width/1.9
  let morphSize = CGSize(
    width: avatarSize.width * 0.85,
    height: avatarSize.height * 1.1)
}
```

这个动画涉及一些数学 - 但不是太多！ 首先，计算化身在相互反弹时应移动的水平距离，并将该值保存在bounceXOffset中。 稍后您将使用变形大小在两个化身碰撞时添加额外的效果。

现在您已知道x偏移，您可以计算化身应移动的位置。 将以下代码添加到searchForOpponent（）：

```swift
let rightBouncePoint = CGPoint(
  x: view.frame.size.width/2.0 + bounceXOffset,
  y: myAvatar.center.y)
let leftBouncePoint = CGPoint(
  x: view.frame.size.width/2.0 - bounceXOffset,
  y: myAvatar.center.y)
```

当化身分别到达左右反弹点时，它们将几乎不会相互接触，此时你将再次使它们彼此远离动画。

最后，将以下代码添加到searchForOpponent（）：

```swift
myAvatar.bounceOff(point: rightBouncePoint,
  morphSize: morphSize)
opponentAvatar.bounceOff(point: leftBouncePoint,
  morphSize: morphSize)
```

bounceOff（point：morphSize :)尚不存在; 你马上就会添加它。 它需要两个参数：化身应该移动的位置和它应该变形的大小。 这就是创建动画所需的全部内容。

打开AvatarView.swift并添加下面的退回方法：

```swift
func bounceOff(point: CGPoint, morphSize: CGSize) {
  let originalCenter = center
  UIView.animate(withDuration: animationDuration, delay: 0.0,
    usingSpringWithDamping: 0.8, initialSpringVelocity: 0.0,
    animations: {
      self.center = point
    },
    completion: { _ in
      //complete bounce to
		} 
	)
}
```

在上面的方法中，首先存储头像视图的中心坐标; 稍后您需要将视图设置为原始位置的动画。 接下来，使用弹簧动画将头像视图移动到bouncePoint坐标。

现在你需要一些东西来动画化身到它的起始位置。 将以下代码添加到bounceOff的末尾（point：morphSize :)：

```swift
UIView.animate(withDuration: animationDuration,
  delay: animationDuration, usingSpringWithDamping: 0.7,
  initialSpringVelocity: 1.0,
  animations: {
    self.center = originalCenter
  },
  completion: { _ in
    delay(seconds: 0.1) {
      self.bounceOff(point: point, morphSize: morphSize)
    }
	} 
)
```

上面的代码使用另一个弹簧动画将头像移回原来的位置。 稍微延迟后，您将从完成关闭再次重新启动动画。

构建并运行项目以查看反弹动画的外观。



[图片]



请注意，当化身视图触摸时，它们会在一起短时间内保持在一起，就好像它们之间存在张力一样。 这个短暂的时期是你使用形状变形技术添加“压扁”效果的地方。

## 变形形状
当两个化身发生碰撞时，它们应该在这种完全弹性的碰撞中稍微挤压。 视图控制器将传递一个变形大小，使角色图像稍微更高，更窄，以达到此效果：



[图片]



这将使它看起来像化身在屏幕中间相遇时相互挤压。

首先要注意的是用于变形效果的框架。 只是为了使事情稍微复杂化，每个化身的框架将有所不同，具体取决于它是从左侧还是右侧动画。

将以下代码添加到bounceOff的底部（point：morphSize :)：

```swift
let morphedFrame = (originalCenter.x > point.x) ?
  CGRect(x: 0.0, y: bounds.height - morphSize.height,
    width: morphSize.width, height: morphSize.height):
  CGRect(x: bounds.width - morphSize.width,
    y: bounds.height - morphSize.height,
    width: morphSize.width, height: morphSize.height)
```

如果化身从左到右动画，则变形化身的最终位置将触摸原始帧的右边缘。 这是从右到左动画化身的完全相反，因为它将最终在原始帧的左边缘。

这意味着头像图像最终会在屏幕中央稍微触摸，如下所示：



[图片]



最后，您可以将形状移动动画代码添加到bounceOff的底部（point：morphSize :)：

```swift
let morphAnimation = CABasicAnimation(keyPath: "path")
morphAnimation.duration = animationDuration
morphAnimation.toValue = UIBezierPath(ovalIn:
  morphedFrame).cgPath
morphAnimation.timingFunction = CAMediaTimingFunction(
  name: .easeOut)
```

在这里，您创建一个CABasicAnimation并将其keyPath设置为path属性。 与任何其他属性图层动画一样，您只需设置结束值，Core Animation将为您呈现中间状态。

然后设置持续时间并使用类UIBezierPath为效果创建椭圆路径。 UIBezierPath非常方便，因为它具有许多方便的初始化方法，包括上面使用的方法，它采用CGRect并创建一个适合rect的椭圆路径。

最后，您将动画设置为缓出，这有助于在化身反弹之前建立紧张感。

这是一个漫长的过程，但剩下的就是将动画添加到头像中，然后才能看到你疯狂的形状转变！

将以下代码行添加到bounceOff的底部（point：morphSize :)：

```swift
circleLayer.add(morphAnimation, forKey: nil)
```

构建并运行项目以查看最终结果：



[图片]



化身框架整齐地相互挤压然后反弹，就像两个愤怒的战斗在前史海中漂浮的阿米巴！

效果并不完全，因为只有帧变形，使图像保持不变。 回想一下，您将头像图像的蒙版设置为CAShapeLayer  - 这与头像框架属于同一类。 所以理论上你可以在你的面具上重复使用框架上的动画对象。

理论在实践中会起作用吗？ 尝试一下，将以下行添加到bounceOff(point: morphSize:)的底部：

```swift
maskLayer.add(morphAnimation, forKey: nil)
```

再次构建和运行您的项目; 这次你应该看到框架和面具变形完美同步：



[图片]



太棒了 - 你刚刚创建了一个非常好看的动画，只有一些数学和一些精心设计的代码块！ 在本章的空白处，您已经学习了如何创建形状和动画，以及如何使用形状图层并将其设置为蒙版。

## 关键点
* 您可以使用CAShapeLayer类在屏幕上绘制动态形状，并设置其笔触和填充颜色，并设置设置形状的路径。
* 您可以通过设置其路径属性的动画来为CAShapeLayer渲染的形状设置动画。
* CAShapeLayer是一种很好的方法，可以通过剪切并在屏幕上显示底层内容的圆形，正方形或星形来剪切其他图层的内容。

## 挑战
本章中的挑战是可选的，但我鼓励您通过它们来练习您的技能并为您的项目添加一些真正的润色。 但是，如果您渴望从渐变动画开始，那么您可以直接进入下一章。

### 挑战1：完成通信状态动画
对于这个挑战，您可以稍微休息一下，因为您可以按照下面的说明操作。 您在此挑战中的任务是添加一些状态消息，以便在您的“搜索对手”任务进展时向用户显示。

打开ViewController.swift并将以下代码添加到searchForOpponent（）的底部：

```swift
delay(seconds: 4.0, completion: foundOpponent)
```

应用程序将在调用foundOpponent（）之前继续“搜索”四秒钟，这将向玩家表明应用程序已找到对手。

将以下方法添加到ViewController类：

```swift
func foundOpponent() {
  status.text = "Connecting..."
  opponentAvatar.image = UIImage(named: "avatar-2")
  opponentAvatar.name = "Ray"
}
```

构建并运行你的项目，大约四秒后，你会看到对手的头像出现：



[图片]



现在该应用程序找到了一个对手，它将开始“连接”两个玩家。 当然，所有这些只是一个模拟，所以你可以构建一个漂亮的动画。

将以下代码添加到foundOpponent（）的底部：

```swift
delay(seconds: 4.0, completion: connectedToOpponent)
```

这会在应用程序显示对手的头像后再插入四秒延迟，之后您将在下一个代码块中调用“连接状态”方法。

将以下新方法添加到类中：

```swift
func connectedToOpponent() {
  myAvatar.shouldTransitionToFinishedState = true
  opponentAvatar.shouldTransitionToFinishedState = true
}
```

上述方法在两个头像视图上将shouldTransitionToFinishedState设置为true。 这将触发您将在下一次挑战中创建的新动画，因此现在不会发生任何事情。

最后，您需要调整最终状态的UI。 将以下代码添加到connectedToOpponent（）的底部：

```swift
delay(seconds: 1.0, completion: completed)
```

此代码为动画提供了第二步，然后调用序列中的最后一步：completed（）。

将最终方法添加到ViewController类：

```swift
func completed() {
  status.text = "Ready to play"
  UIView.animate(withDuration: 0.2) {
    self.vs.alpha = 1.0
    self.searchAgain.alpha = 1.0
  }
}
```

completed（）将屏幕顶部的状态消息设置为“Ready to play”，然后淡入“vs.”标签和“再次搜索”按钮以重新启动动画序列。 该按钮已连接到actionSearchAgain（），因此您可以点击它以重新启动动画。

构建并运行以查看整个动画序列：



[图片]



### 挑战2：将角色变成正方形
在这一点上，化身只是永远地弹跳。 一旦游戏连接到对手，您就想停止动画并反映UI中的状态变化。

在此挑战中，您将使用头像类的shouldTransitionToFinishedState属性; 当它设置为true时，你将打破反弹动画并将头像变形为方形。

打开AvatarView.swift并添加一个名为isSquare的新变量，并将其初始值设置为false。 滚动到bounceOff（点：morphSize :)并找到动画完成块，其中有一个递归调用bounceOff（point：morphSize :)。

您需要将该调用包装在条件中，以便仅在isSquare仍为false时运行。

接下来，当化身最后一次互相反弹时，你会运行一个额外的动画。

找到  //完全退回以发表评论并将其替换为以下内容：

```swift
if self.shouldTransitionToFinishedState {
    self.animateToSquare()
}
```

最后，将animateToSquare（）方法添加到AvatarView并编写代码以执行以下操作：

1. 将isSquare设置为true。
2. 使用avatar的bounds矩形使用UIBezierPath（rect :)创建Bezier路径，并将此bezier路径的CGPath存储在名为squarePath的常量中。
3. 使用路径的键路径创建新的图层动画，并将其持续时间设置为0.25秒。
4. 将动画的fromValue设置为circleLayer.path，将toValue设置为squarePath。 这定义了从圆形到方形的变形动画。
5. 将动画添加到circleLayer，然后将其path属性设置为squarePath。
6. 同样，将动画添加到遮罩层并将其path属性设置为squarePath。

构建并运行您的项目; 最后的反弹看起来会更加漂亮。 虽然头像最后一次触摸它们仍然看起来像这样：



[图片]



它们会反弹回到它们的起点，变成正方形以获得惊人的效果：

到目前为止，您已了解使用形状图层和蒙版的基本动画技术。 您可能已经想过很多方法在自己的应用程序中应用形状动画。

您将继续使用第15章“笔画和路径动画”中的形状和CAShapeLayer动画。继续第14章“渐变动画”，学习如何使用动画渐变为动画添加一些非常整洁的效果。