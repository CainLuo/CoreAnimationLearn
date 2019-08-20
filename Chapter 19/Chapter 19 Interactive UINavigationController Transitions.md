# Chapter 19: Interactive UINavigationController Transitions

您在上一章中创建的揭示过渡看起来非常整洁，但自定义动画只是故事的一半。 亲爱的朋友，你已经避开了真相，但没有了; 作为你通过本书走向自己的方式的奖励，你即将熟悉iOS古人的秘密。

您不仅可以为转换创建自定义动画 - 还可以使其交互并响应用户的操作。

通常，您通过平移手势驱动此操作，这是您将在本章中采用的方法。

完成后，您的用户可以通过在屏幕上滑动手指来浏览显示过渡。 这会有多酷？

是的，我以为你会感兴趣！ 继续阅读，了解它是如何完成的！

## 创建交互式过渡
当导航控制器向其代表询问动画控制器时，可能会发生两件事。 您可以返回nil，在这种情况下，导航控制器会运行标准过渡动画。 你已经知道了很多。

但是 - 如果你确实返回一个动画控制器，那么导航控制器会向其代表询问交互控制器，如下所示：



[图片]



交互控制器根据用户的操作移动过渡，而不是简单地从开始到结束动画更改。 交互控制器不一定需要是与动画控制器分开的类; 事实上，当两个控制器属于同一个类时，执行某些任务会更容易一些。 您只需要确保所述类符合UIViewControllerAnimatedTransitioning和UIViewControllerInteractiveTransitioning。

UIViewControllerInteractiveTransitioning只有一个必需的方法 -  startInteractiveTransition（_ :)  - 它将转换上下文作为其参数。 然后，交互控制器定期调用updateInteractiveTransition（_ :)来移动转换。 首先，您需要更改处理用户输入的方式。

## 处理平移手势
首先，MasterViewController中的轻敲手势识别器不会再削减它了。 水龙头瞬间发生，然后就消失了; 您无法跟踪其进度并使用它来推动转换。 另一方面，平移手势具有转换的开始，进展和结束阶段的明确状态。

打开本章的入门项目; 或者，您可以使用上一章中已完成的项目（包括挑战）。

打开Main.storyboard; 转到主视图控制器并将标签文本更改为屏幕底部“滑动启动”，如下所示：



[图片]



这将反映您期望用户执行的操作。

接下来，打开MasterViewController.swift并从viewDidAppear（_ :)中删除以下代码：

```swift
let tap = UITapGestureRecognizer(target: self, action: #selector(didTap))
view.addGestureRecognizer(tap)
```

在其位置，插入以下平移识别器代码：

```swift
let pan = UIPanGestureRecognizer(target: self, action:
#selector(didPan(_:)))
view.addGestureRecognizer(pan)
```

当用户跨越屏幕时，识别器会在MasterViewController类上调用didPan（_ :)。

要摆脱错误，立即在Xcode中显示，向MasterViewController添加一个空的didPan（_ :)方法：

```swift
@objc func didPan(_ recognizer: UIPanGestureRecognizer) {
}
```

你需要修改你的RevealAnimator类来处理新的交互式转换; 你将在下一节中解决这个问题。

## 使用交互式动画师课程
要管理您的过渡，您将使用Apple的内置交互式动画师类之一：UIPercentDrivenInteractiveTransition。 此类符合UIViewControllerInteractiveTransitioning，并允许您将转换的进度设置为表示完成百分比的值。

这使您的生活更轻松，因为您可以使用此类相应地调整percentComplete属性并调用update（）来设置转换的当前可见进度。 这将跳过过渡动画到与计算的转换进度相对应的点。 在本章的其余部分中，您将了解有关UIPercentDrivenInteractiveTransition如何工作的更多信息。

打开RevealAnimator.swift并更新文件顶部的类定义，如下所示：

```swift
class RevealAnimator: UIPercentDrivenInteractiveTransition,
UIViewControllerAnimatedTransitioning, CAAnimationDelegate {
```

请注意，UIPercentDrivenInteractiveTransition是一个类而不是其他协议，所以它需要处于第一位置。 现在RevealAnimator继承自UIPercentDrivenInteractiveTransition。

接下来，添加以下属性以告知动画师是否应以交互方式驱动转换：

```swift
var interactive = false
```

现在将以下方法添加到RevealAnimator：

```swift
func handlePan(_ recognizer: UIPanGestureRecognizer) {
}
```

当用户在屏幕上平移时，您将把识别器传递给RevealAnimator中的handlePan（_ :)，此时您将更新当前的过渡进度。 您将稍微填充handlePan（_ :)，但首先您需要设置手势处理。

打开MasterViewController.swift并添加以下委托方法，为该文件中的MasterViewController扩展提供交互控制器：

```swift
 func navigationController(_
  navigationController: UINavigationController,
	interactionControllerFor animationController:
  UIViewControllerAnimatedTransitioning) ->
  UIViewControllerInteractiveTransitioning? {
  if !transition.interactive {
return nil
}
  return transition
}
```

只有在希望转换为交互式时才返回交互式控制器。 例如，在您的Logo Reveal项目中，显示转换是交互式的，但自定义弹出转换将保持原样。

现在，您需要将平移手势识别器连接到交互控制器。 在MasterViewController中找到didPan（_ :)并替换为：

```swift
@objc func didPan(_ recognizer: UIPanGestureRecognizer) {
  switch recognizer.state {
  case .began:
    transition.interactive = true
    performSegue(withIdentifier: "details", sender: nil)
  default:
    transition.handlePan(recognizer)
  }
}
```

当平移手势开始时，您确保交互式设置为true，然后开始segue到下一个视图控制器。 执行segue将启动过渡，如前一章所述; 您为动画控制器和交互控制器添加的委托方法返回转换。

在所有情况下，如果手势已经开始，您只需将操作交给交互控制器，如下所示：



[图片]



## 计算动画的进度
平移手势处理程序中最重要的一点是确定过渡应该有多远。

打开RevealAnimator.swift并将以下代码添加到handlePan（）：

```swift
let translation = recognizer.translation(in:
  recognizer.view!.superview!)
var progress: CGFloat = abs(translation.x / 200.0)
progress = min(max(progress, 0.01), 0.99)
```

首先，您从平移手势识别器获得翻译;翻译让您知道用户在X轴和Y轴上移动手指/手写笔/附件/其他任何点数。从逻辑上讲，用户从初始位置越远，转换的进度就越大。

要计算当前进度，请在X轴上进行平移并将其除以200点。例如，如果用户的手指离初始平移位置100个点，则转换将完成50％。 200点是一个任意数字，但它是用户需要平移以完成转换的总距离的良好起点。您不应该关心用户是向右还是向左平移 - 这就是您使用abs（）获取平移距离的绝对值的原因。

最后，将进度变量限制在0.01和0.99之间;我的测试显示，如果您不让用户完成或仅从平移手势恢复转换，则交互控制器会表现得更好。

现在您已了解过渡动画的进度，也可以更新过渡动画。

将以下代码添加到handlePan（）：

```swift
switch recognizer.state {
  case .changed:
    update(progress)
  default:
		break
}
```

update（）是UIPercentDrivenInteractiveTransition的一个方法，它设置过渡动画的当前进度。

当用户在屏幕上平移时，手势识别器会在MasterViewController中重复调用didPan（），然后在RevealAnimator中将识别器转发到handlePan（）。

不幸的是，如果你在这个时候建立和运行，你会看到一些动画似乎跟随你的手势而其他动画只是按照自己的节奏运行。 UIPercentDrivenInteractiveTransition与图层动画的效果不如查看动画那么好，所以你必须做一些额外的工作。

首先，将此属性添加到RevealAnimator：

```swift
private var pausedTime: CFTimeInterval = 0
```

现在，通过将以下代码添加到animateTransition的开头（使用:)来控制图层：

```swift
if interactive {
  let transitionLayer = transitionContext.containerView.layer
  pausedTime = transitionLayer.convertTime(CACurrentMediaTime(), from:
nil)
  transitionLayer.speed = 0
  transitionLayer.timeOffset = pausedTime
}
```

你在这里做的是阻止图层运行自己的动画。 这将冻结所有子图层动画。 现在覆盖update（_ :)以将图层与动画一起移动：

```swift
override func update(_ percentComplete: CGFloat) {
  super.update(percentComplete)
  let animationProgress = TimeInterval(animationDuration) *
TimeInterval(percentComplete)
  storedContext?.containerView.layer.timeOffset = pausedTime +
animationProgress
}
```

在这里，您要计算动画的距离并相应地设置图层的timeOffset，这会将动画移动到时间轴中的适当点。

构建并运行您的项目; 在屏幕上平移，看看你的过渡是什么样的：



[图片]



由于您的转换不完整，因此只要您抬起手指，整个导航就会中断。 但是，您可以看到揭示动画跟随您的平移手势 - 您即将完成交互式转换！

> 注意：感觉图层动画的额外工作是UIKit错误; 如果你没有在过渡中使用图层动画，你不必做所有这些乱七八糟的事情，但是在过程中，转换正在做同样的事情，以使视图动画来回擦洗。

剩下的就是处理交互式转换的最终状态。

## 处理提前终止
在这里，您面临一个全新的问题：用户可能会在X轴上平移200点之前抬起手指。 这使得过渡处于未完成状态。

幸运的是，UIPercentDrivenInteractiveTransition为您提供了几种免费方法，您可以根据用户的操作来恢复或完成转换。

在上面添加的switch语句中添加以下两种情况，就在默认情况之前：

```swift
case .cancelled, .ended:
  if progress < 0.5 {
    cancel()
  } else {
		finish() 
  }
```

就您的项目而言，.cancelled和.ended案例实际上是相同的。 在任何一种情况下，如果用户在释放之前摇摄得足够远，则完全呈现新的视图控制器; 如果没有，您想要回滚动画进度。

如果用户平移的距离小于所需距离的50％，则调用cancel（） - 一种继承的方法 - 将转换动画设置回其初始状态。 如果用户平移了超过50％的距离，则调用finish（），在剩下的时间内播放动画。 这两种状态如下所示：



[图片]



因为你正在使用图层动画，所以这里还有一些工作要做。 记住你冻结了图层并手动更新了它; 当您取消或完成转换时，您需要取消冻结它。 像这样覆盖cancel（）和finish（）：

```swift
override func cancel() {
  restart(forFinishing: false)
  super.cancel()
}

override func finish() {
  restart(forFinishing: true)
  super.finish()
}

private func restart(forFinishing: Bool) {
  let transitionLayer = storedContext?.containerView.layer
  transitionLayer?.beginTime = CACurrentMediaTime()
  transitionLayer?.speed = forFinishing ? 1 : -1
}
```

取消时，需要将图层设置为向后运行，因此速度设置为-1。

构建并运行应用程序，并尝试部分平移和大部分方式来查看差异。

您是否注意到当您浏览揭示动画时，您无法返回列表？ 那是因为你在handlePan（_ :)中将interactive设置为true，而你永远不会将它重置为false！ 因此，当您弹出视图控制器时，您将从委托方法返回一个永远不会更新的交互控制器 - 并且您的弹出转换将停留在0％进度。

重置交互属性的正确位置是平移手势结束时。

将以下代码添加到.cancelled，.ended案例中：

```swift
interactive = false
```

他应该让你回到初始屏幕。

再次构建并运行，您应该能够来回移动。



[图片]



您已完成Logo Reveal项目。 RevealAnimator现在正在执行两种不同的过渡动画，包括一种互动动画！ 很好！

这包含了视图控制器转换的部分。 恭喜您完成本节的工作; 这些过渡API并不容易学习，但所有的努力都是值得的。

## 关键点
* 通过在过渡动画中采用UIPercentDrivenInteractiveTransition协议，您可以轻松地为自定义过渡添加交互性。
* 交互式过渡通常由用户手势驱动。 UIPanGestureRecognizer是一个方便的课程，可以为您提供持续的手势反馈。
* 您可以通过在UIPercentDrivenInteractiveTransition上设置交互属性的值来在交互式和非交互式转换模式之间切换。

## 挑战
最后的挑战比平时更难，但现在你是一个动画忍者，我知道你可以处理我扔给你的任何东西！

### 挑战1：使流行过渡互动

您在此挑战中的任务是使流行过渡成为交互式。 这并不像听起来那么容易，因为您需要在整个项目中的许多地方更改代码。

下面的挑战方向只是广泛的描述，因此您需要在开始编码之前规划您的方法。

首先，在DetailViewController中; 创建一个弱属性来保存动画师并从MasterViewController获取动画对象。 您可以从导航控制器堆栈访问MasterViewController。

完成后，向DetailViewController添加一个平移手势处理程序。 你的处理程序应该与MasterViewController中的方法几乎完全相同，只有一个小区别：它应该弹出当前的视图控制器而不是调用segue。

此时，自定义弹出过渡应该主要是功能性的。 您正在使用视图动画进行弹出过渡，但视图动画不需要调整beginTime或completionSpeed。 确保在动画操作为.pop时不修改这些属性。

您将最终获得一个很酷的交互式流行转换 - 并且不要忘记确保点击导航栏中的“后退”按钮仍然有效！