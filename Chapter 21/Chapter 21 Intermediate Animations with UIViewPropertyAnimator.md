# Chapter 21: Intermediate Animations with UIViewPropertyAnimator

您已经使用UIViewPropertyAnimator尝试了一些动画，并且已经开始通过向界面添加令人愉快的动画来改进Widgets项目用户体验。 您还研究了创建基本动画和关键帧动画，并发现使用UIViewPropertyAnimator类并不困难！

更重要的是，当使用UIView.animate（withDuration：...）API集时，你已经解决了一些不那么简单的问题 - 例如，检查动画当前是否正在运行，有条件地添加动画和完成，以及抽象 动画进入独立课程。

如果您成功完成了上一章中的挑战，只需重新打开项目并继续处理。 否则，您可以使用本章提供的入门项目：



[图片]



让我们看看你如何通过使用自定义时序给你的动画这个额外的东西！

## 自定义动画计时
在本书中，您一直在使用四种内置曲线：线性，易用性，易用性和易用性。 到目前为止，对于那些尚未说过的内容没什么好说的，所以通过本章的大部分内容，您将专注于自定义曲线。 如果有任何机会，你跳过了解释内置曲线的前几章，请快速阅读第1章“视图动画入门”并搜索动画缓动部分。

假设您已经掌握了四条内置曲线的内容，那么让我们来看一下动画中的曲线。

### 内置定时曲线
目前，当您激活搜索栏时，您会在窗口小部件顶部的模糊视图中淡入淡出。 在此示例中，您将删除该淡入淡出动画并为模糊效果本身设置动画。

打开LockScreenViewController.swift并向该类添加一个新方法:

```swift
func blurAnimations(_ blurred: Bool) -> () -> Void {
  return {
	} 
}
```

这是一个返回准备好的动画块的方法，您可以稍后将其添加到动画师中。 根据参数值，块中的动画将在blurView中删除或创建模糊效果。

首先让我们添加代码来创建动画。 在大括号之间插入：

```swift
self.blurView.effect = blurred ?
  UIBlurEffect(style: .dark) : nil
self.tableView.transform = blurred ?
  CGAffineTransform(scaleX: 0.75, y: 0.75) : .identity
self.tableView.alpha = blurred ? 0.33 : 1.0
```

将blurView上的效果属性设置为nil或模糊效果将生成动画。 此外，您还可以调整包含窗口小部件的表视图的可见性和变换。

很好！ 现在，完成的方法如下所示：

```swift
func blurAnimations(_ blurred: Bool) -> () -> Void {
  return {
    self.blurView.effect = blurred ? UIBlurEffect(style: .dark) : nil
    self.tableView.transform = blurred ?
      CGAffineTransform(scaleX: 0.75, y: 0.75) : .identity
    self.tableView.alpha = blurred ? 0.33 : 1.0
	} 
}
```

您可以使用它根据屏幕的当前状态生成两个不同的动画。

接下来，您需要调整UI。 滚动到viewDidLoad（）并删除以下两行：

```swift
blurView.effect = UIBlurEffect(style: .dark)
blurView.alpha = 0
```

这些是设置效果的线条，但您不再需要它们，因为您将使用新的基于效果的动画。

说到这个，用以下内容替换toggleBlur（_ :)的内容：

```swift
func toggleBlur(_ blurred: Bool) {
  UIViewPropertyAnimator(duration: 0.55, curve: .easeOut,
    animations: blurAnimations(blurred))
    .startAnimation()
}
```

请注意，curve参数的类型为UIViewAnimationCurve。 此枚举包括四种内置曲线类型：.linear，.easeIn，.easeOut和easeInOut。

尝试新动画; 点击搜索栏，您会看到模糊效果逐渐显示在屏幕上：



[图片]



注意模糊不仅仅是淡入或淡出，而是实际插入效果视图中的模糊量。

现在你可以尝试不同的时序曲线，如.easeIn或.linear，看看它们如何改变动画。

> 注意：模拟器中的模糊动画可能有点不稳定，要享受iOS动画的强大功能，始终享受设备的最终效果。

### 自定义Bézier曲线
有时当你想要非常具体地说明动画的时间时，使用这些曲线只是“开始时减速”或“慢慢结束”是不够的。

在本书的前面，您了解到可以创建自定义CAMediaTimingFunction并为您的图层动画定制自定义时序。 但是，在一个说明中简要提到了这一点，你没有机会更多地了解它。

在本节中，您将了解Bézier曲线是什么以及如何使用它们来设计您自己的自定义动画时序。 好消息是因为UIViewPropertyAnimator在幕后使用图层动画，您可以返回并将本章中掌握的内容应用到图层动画中。

但首先 - 贝塞尔曲线是什么？

> 注意：如果您已经对Bézier曲线有了深入的了解，请跳过对它们背后的机制的相当简单的解释。

让我们从简单的事情开始 - 一条线。 它非常整洁，因为你需要在屏幕上画一条线就是定义它的两个点的坐标; 开始（A）和结束（B）：



[图片]



能够如此精确地在屏幕上描述形状的便利方面是您还可以对其应用变换：您可以缩放线条，移动线条以及旋转线条。 一切都归功于坐标系中的这两点。 此外，您可以将行保留到磁盘并将其加载回来，因为您可以使用数字来描述它们，并且您知道如何保留它们！

现在让我们来看看曲线。 曲线比线条更有趣，因为它们可以在屏幕上绘制任何东西。 例如：



[图片]



你在上面看到的是四条曲线放在一起; 他们的两端在你看到小白方块的地方相遇。 图中有趣的是小绿圈。 看完图片一段时间后，你会注意到每个曲线弯曲的圆圈“有点定义”。

所以曲线不是随机的。 它们也有一些细节，就像线条一样，可以帮助你通过坐标定义它们。 然后，您可以获得坐标的所有好处，例如持久化，转换它们等。

您可以通过向线添加控制点来定义曲线。 让我们在之前的行中添加一个控制点：



[图片]



您可以想象由连接到线的铅笔绘制的曲线，其起点沿着AC线移动，其终点沿着线CB移动：



[图片]



具有一个控制点的Bézier曲线称为二次曲线。 但是，你对立方Bézier曲线更感兴趣 - 那些有两个控制点。

您还可以使用三次曲线来描述动画时序。 实际上，您使用的内置曲线也是为您预定义的三次曲线。

Core Animation使用始终以坐标（0,0）开始的三次曲线，它表示动画持续时间的开始。 当然，这些时序曲线的终点始终是（1,1） - 动画的持续时间和进度的结束。

让我们来看看轻松曲线：



[图片]



随着时间的推移（在坐标空间中从左向右水平移动），曲线在垂直轴上的进展非常小，然后在动画持续时间的一半左右，进度加快并随着时间的推移而赶上，因此它们都达到了 （1,1）在动画结束时。

这一切都有意义，对吧？

你能猜出下面哪一个是缓出的，哪一个是一个轻松的曲线？



[图片]



现在您已了解Bézier曲线的工作原理，唯一剩下的问题是如何在视觉上设计一些曲线并获得控制点的坐标，以便您可以将它们用于iOS动画。

> 注意：打开Web浏览器并访问http://cubic-bezier.com。 这是计算机科学研究员和演讲者Lea Verou的便利网站。 它允许您拖动立方Bézier的两个控制点并查看即时动画预览。



[图片]



我鼓励你一起玩，直到你知道不同的时序曲线如何影响动画时序。

接下来，您将继续向Widgets项目添加自定义计时动画.

打开LockScreenViewController.swift并将toggleBlur（）中的现有动画替换为：

```swift
func toggleBlur(_ blurred: Bool) {
  UIViewPropertyAnimator(duration: 0.55,
    controlPoint1: CGPoint(x: 0.57, y: -0.4),
    controlPoint2: CGPoint(x: 0.96, y: 0.87),
    animations: blurAnimations(blurred))
    .startAnimation()
}
```

这是UIViewPropertyAnimator类的第二个便利初始值设定项。 除了持续时间和动画块之外，还需要两个控制点作为参数。 这些可以帮助您定义自定义三次曲线。

等一等！ 上面的一点有一个负坐标！

确实！ 既然你无论如何做自定义时序曲线，为什么不做异国情调呢？

您可以拖动控制点，以便将曲线拉入定义进度轴负值的空间。 效果相当有趣：如果你正在从A点向B点的正确方向移动视图，它将首先向左“退一步”，然后继续朝着B点的正确方向。

在模糊和缩放动画的情况下，表格视图实际上会在缩小之前首先缩放一点。 这样做会为动画添加一点“弹性”。

但请注意：如果你过度使用它，这将对你的动画产生一个相当滑稽的效果。

### 弹簧动画
还有另一个便利初始化器--UIViewPropertyAnimator（duration：dampingRatio：animations :)  - 用于定义弹簧驱动的动画。

这将产生与UIView的动画相同的动画（withDuration：delay：usingSpringWithDamping：initialSpringVelocity：options：animations：
完成:)，初始速度为0。

与UIView方法一样，此API会向后创建弹簧动画，如第11章所述。您可以提供动画的持续时间，UIKit会计算弹簧的所有方面，以便为您提供持续时间。 你知道这并没有像正确计算那样给出一个好的弹簧效果。 幸运的是，有一种更好的方法可以使用UIViewPropertyAnimator创建弹簧动画。

### 定制供应商
满足你将在这里讨论的第四个也是最后一个初始化程序：UIViewPropertyAnimator（duration：timingParameters :)。

这一次，您可以创建一个可以为动画提供任何计时数据的全新对象！您可以使用其中一个UIKit对象来定义自定义立方体或基于弹簧的计时，但您也可以自行推出。

您将看到如何创建自定义弹簧动画，然后再转到本章的下一部分，您将在实践中创建一些弹簧动画。

名为timingParameters的第二个参数是UITimingCurveProvider类型 - 由UIKit定义的协议。 UIKit中有两个符合该协议的类：UICubicTimingParameters和UISpringTimingParameters。

我们来看看UISpringTimingParameters。

#### 提供阻尼和速度

即使您使用的是自定义定时提供程序，您仍然可以选择简单的方法，并提供阻尼比和初始速度，就像使用便捷初始化程序时一样。代码如下所示：

```swift
let spring = UISpringTimingParameters(dampingRatio:0.5,
  initialVelocity: CGVector(dx: 1.0, dy: 0.2))
let animator = UIViewPropertyAnimator(duration: 1.0,
  timingParameters: spring)
```

spring参数表示弹簧的配置，并将其提供给动画对象以用于动画的计时。 如前所述，这仍将计算弹簧“向后”。

注意初始速度是矢量类型。 UIKit将在开始时应用二维初始速度，以防您为任何视图的位置或大小设置动画。 如果您要为视图的位置设置alpha或单个轴的动画，UIKit将仅考虑初始速度矢量的dx属性。

initialVelocity也是一个可选参数，因此如果您根本不需要设置速度，只需提供阻尼比即可。

#### 定制弹簧
如果您想更加具体地了解弹簧，可以在UISpringTimingParameters上使用不同的初始化器，让您指定弹簧的质量，刚度和阻尼，就像您在本书前面对图层动画所做的那样。

因此，配置自定义弹簧的代码如下：

```swift
let spring = UISpringTimingParameters(mass: 10.0,
  stiffness: 5.0, damping: 30,
  initialVelocity: CGVector(dx: 1.0, dy: 0.2))
let animator = UIViewPropertyAnimator(duration: 1.0, t
  imingParameters: spring)
```

如果您需要快速了解所有这些参数的工作原理，请快速阅读第11章“图层弹簧”。

在下一节中，您将尝试一些自定义时序动画。

## 自动布局动画
唷！ 这是本章的一个相当冗长的理论部分，所以我很高兴你能编写一些代码并尝试一些动画。

您已经在第7章“动画约束”中熟练掌握了自动布局动画，所以在下一部分中您将要制作一些约束动画并不会让您感到意外。

使用UIViewPropertyAnimator的布局约束动画与使用UIView.animate（withDuration：...）创建它们的方式非常相似。 诀窍是更新约束，然后从动画块中调用layoutIfNeeded（）。

让我们尝试使用UIViewPropertyAnimator。

打开AnimatorFactory.swift并添加一个新的工厂方法：

```swift
@discardableResult
static func animateConstraint(view: UIView, constraint:
  NSLayoutConstraint, by: CGFloat) -> UIViewPropertyAnimator {
}
```

在此方法中，您将为约束的常量设置更改动画，然后在提供的视图上调用layoutIfNeeded（）。

这个动画看起来像什么？ 不知道！ 这实际上取决于您为方法提供的约束类型。 如果给它一个尾随空格约束，它将水平移动视图; 如果为其提供高度约束，则视图将向上或向下缩放。

在方法内部创建一个动画师：

```swift
let spring = UISpringTimingParameters(dampingRatio: 0.2)
let animator = UIViewPropertyAnimator(duration: 2.0,
  timingParameters: spring)
animator.addAnimations {
  constraint.constant += by
  view.layoutIfNeeded()
}
return animator
```

您使用简单的方便弹簧初始化程序，然后您只需更改约束并触发AutoLayout传递。

现在切换到LockScreenViewController.swift并添加到viewWillAppear（_ :)：

```swift
dateTopConstraint.constant -= 100
view.layoutIfNeeded()
```

这会将日期标签向上移动100点，如下所示：



[图片]



接下来，触发动画以将标签（以及附加到其上的所有其他视图）向下移动到其原始位置。

将以下内容附加到viewDidAppear（_ :)：

```swift
AnimatorFactory.animateConstraint(view: view,
  constraint: dateTopConstraint, by: 100)
  .startAnimation()
```

运行应用程序并查看生成的动画！ 由于您正在向下移动所有视图并扩展表视图，因此会导致相当复杂而平滑的过渡：



[图片]



虽然动画看起来有些过分。 前几次，它看起来不错，但如果您的用户每天多次看到这种有弹性的过渡，他们肯定会讨厌您的应用。

这是一个很好的提醒，UI弹簧动画应该是关于适度的。

切换回AnimatorFactory.swift并更改弹簧动画的参数。 将dampingRatio设置为0.55，持续时间设置为1.0。 这应该使动画更微妙但仍然有趣。

接下来，当您为约束设置动画时，您将探索不同的情况。 目前，当您点击“显示更多”时，窗口小部件会更改其高度约束并重新加载顶部表视图内容以调整窗口小部件的大小。

在下一个动画中，您将为单元格高度更改设置动画。

打开WidgetCell.swift文件，找到toggleShowMore（_ :)。 您可以查看内部以查看更改单元格高度的当前代码并重新加载父表视图。

您将完全重建此方法。 删除其中的所有代码toggleShowMore（_ :)并将其替换为：

```swift
self.showsMore = !self.showsMore
```

首先让我们定义您想要运行的动画。 附加到toggleShowMore（_ :)：

```swift
let animations = {
  self.widgetHeight.constant = self.showsMore ? 230 : 130
  if let tableView = self.tableView {
    tableView.beginUpdates()
    tableView.endUpdates()
    tableView.layoutIfNeeded()
	} 
}
```

在这段代码中，你执行一个小技巧。 首先，您照常更改约束，但随后在表视图上调用beginUpdates（）和endUpdates（）。 这样做会询问所有细胞的高度，并根据需要调整布局。 如果您的任何单元格表示它想要更高或更短，UIKit将相应地调整其框架。

在块结束时，调用layoutIfNeeded（）以确保布局更改将在动画块内发生。

现在让我们创建动画师。 添加到toggleShowMore（_ :)的末尾：

```swift
let spring = UISpringTimingParameters(mass: 30, stiffness: 1000,
  damping: 300, initialVelocity: CGVector(dx: 5, dy: 0))
toggleHeightAnimator = UIViewPropertyAnimator(duration: 0.0,
timingParameters: spring)
toggleHeightAnimator?.addAnimations(animations)
toggleHeightAnimator?.startAnimation()
```

该视图已经具有一个名为toggleHeightAnimator的属性，因此您只需创建一个弹簧配置并将新的动画制作工具存储在该属性中。 请注意，在这种情况下，您可以定义所有弹簧属性，因此将忽略传递给属性动画制作工具的持续时间。 弹簧本身决定了动画运行的时间。

运行应用程序，点击“显示更多”，享受平滑的弹簧驱动动画：



[图片]



在方法的底部，添加以下代码以重新加载窗口小部件中的图标：

```swift
widgetView.expanded = showsMore
widgetView.reload()
```

此代码在集合视图上触发reloadData（）并重新加载所有图标。 再次尝试动画，这次根据小部件高度看到不同数量的图标：



[图片]



## 内置视图过渡
要完成此动画，您将看到使用UIViewPropertyAnimator的内置视图过渡。

在第3章“过渡”中，您了解了可以在iOS中使用的内置视图过渡。 现在，您将使用交叉淡入淡出将窗口小部件按钮的标题从“显示更多”更改为“显示更少”，反之亦然。

在toggleShowMore（_ :)里面，在定义动画块后（但在定义动画师之前）添加此代码：

```swift
let textTransition = {
  UIView.transition(with: sender, duration: 0.25,
    options: .transitionCrossDissolve,
    animations: {
      sender.setTitle(
        self.showsMore ? "Show Less" : "Show More",
        for: .normal)
},
    completion: nil
  )
}
```

您可以定义视图过渡，并在其动画块内部根据窗口小部件当前是否已扩展来更改按钮标题。

那么如何将这种过渡添加到您的动画师？ 就像你对任何其他动画一样阻止！

找到向动画师添加动画的位置，并添加textTransition以及0.5延迟因子。 最终代码应如下所示：

```swift
toggleHeightAnimator?.addAnimations(animations)
toggleHeightAnimator?.addAnimations(textTransition, delayFactor: 0.5)
toggleHeightAnimator?.startAnimation()
```

这将改变按钮标题，具有很好的交叉淡入淡出效果：



[图片]



如果您想尝试其他转换，请使用.transitionFlipFromTop替换.transitionCrossDissolve  - 这是我个人的最爱。 这种转换使得更新按钮标题看起来更加奇特。

你开始将UIViewPropertyAnimator推向极限了！ 在继续下一章和交互式动画之前，请务必查看本章的挑战，该挑战将向您介绍使用UIViewPropertyAnimator的创意添加动画。

## 关键点
* 使用UIViewPropertyAnimator，您可以为UIBlurEffect值设置动画并创建令人印象深刻的模糊动画。
* 您可以使用带有UIViewPropertyAnimator的两个控制点自定义Bézier曲线创建自己的自定义动画计时并远离预定义的缓动常量。
* 最后但并非最不重要的是，您可以将预定义的UIKit转换API（例如：duration：options：animations：completion）与其他UIViewPropertyAnimator动画混合使用，它们将“一起工作”。

## 挑战
### 挑战1：添加动画
使用UIView.animate（withDuration：...）时，将动画添加到同一视图属性中会相加。
例如，如果您要将屏幕上的视图从A点移动到B点，并在终点中途改变主意并决定将视图发送到C点，那么它将不会中断 当前点并直接向新的终点移动。



[图片]



UIKit比这更聪明，所以默认情况下它会尝试将你的观点“放松”到新的轨迹中。 实际的动作看起来像这样：



[图片]



添加更改时动画不会被替换，但会合并以便更改发生更改。

UIViewPropertyAnimator也可以处理这些情况（不同时序提供商的成功不同）。

对于此挑战，如果在上一个动画完成之前在toggleShowMore（_ :)中再次切换菜单状态，则应该将“动画”添加到现有的toggleHeightAnimator，而不是创建新动画师。

* 使用isRunning检查toggleHeightAnimator当前是否正在运行。
* 如果已经创建了动画师，请暂停它，添加新的动画块，然后继续动画！

完成此挑战后，请转到下一章，了解如何为属性动画制作动画添加交互性。